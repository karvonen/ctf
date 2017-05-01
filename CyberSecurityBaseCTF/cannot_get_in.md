### Cannot get in
>One of our cyber m0nkeys has forgotten his nix password. He is a fantasy fan so the password is likely a character in either LotR or GoT. That leaves us with roughly 2800 characters with first and last names to try at least and that is if he has not tried to be sneaky with the names. Can you help our monkey? Here is the unshadowed information "cybersecbase:$6$LwmDTb98$dAMmGCkiIakUVtT.bhYujjHGAwCd3un9KdYwfEDdJef/H9Q62mKFpOIA84.W0yDOiXKr4T7Gwpgw2JjD.4yGK.:1000:1000:csb,,,:/home/cybersecbase:/bin/bash"

I hadn't done password cracking before but I knew in principle how things worked. The `0` of `m0nkeys` in the description was probably a hint but I didn't notice that until completing the challenge.

First task was to hastily google for some LotR and GoT character names and throw them in to a file. It made sense to remove all spaces from the names although looking back it would have been better to include all combinations: 'Firstname', 'Lastname' and 'FirstnameLastname'. Luckily the password ended up being 'FirstnameLastname'. The wordlist that was used is [here](https://github.com/karvonen/ctf/blob/master/CyberSecurityBaseCTF/files/wordlist).

The rule list used was 'dive' from [hashcat's github repo](https://github.com/hashcat/hashcat/blob/master/rules/dive.rule).

```
hashcat64.exe -m 1800 -a 0 -o cracked.txt passwd wordlist -r dive.rule

Session..........: hashcat
Status...........: Cracked
Hash.Type........: sha512crypt $6$, SHA512 (Unix)
Hash.Target......: $6$LwmDTb98$dAMmGCkiIakUVtT.bhYujjHGAwCd3un9KdYwfED....4yGK.
Time.Started.....: Thu Apr 27 04:13:39 2017 (2 mins, 46 secs)
Time.Estimated...: Thu Apr 27 04:16:25 2017 (0 secs)
Guess.Base.......: File (wordlist)
Guess.Mod........: Rules (dive.rule)
Guess.Queue......: 1/1 (100.00%)
Speed.Dev.#1.....:    47155 H/s (0.44ms)
Recovered........: 1/1 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 31466318/388218948 (8.11%)
Rejected.........: 23681554/31466318 (75.26%)
Restore.Point....: 0/3918 (0.00%)
Candidates.#1....: Adalbert3olger -> Zollo
HWMon.Dev.#1.....: Temp: 74c Fan: 33% Util: 89% Core:1885MHz Mem:5005MHz Bus:16

E:\Downloads\hashcat-3.5.0>type cracked.txt
$6$LwmDTb98$dAMmGCkiIakUVtT.bhYujjHGAwCd3un9KdYwfEDdJef/H9Q62mKFpOIA84.W0yDOiXKr4T7Gwpgw2JjD.4yGK.:RyellaFr3y
```
Cracking the password took 2m 46s using an overclocked GTX 1080. Flag was `RyellaFr3y`.

