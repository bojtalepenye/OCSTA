# Offline Credential Stuffing Attack

<p align="center">
  <img src="https://github.com/user-attachments/assets/62578142-f120-42c0-8082-de9a7ba6a430" alt="Logo" width="260" height="260" />
</p>

# Overview
This script allows you to correlate known `username/email:hash:password` pairs with `username:hash` pairs, producing an output in the format `username:hash:password`.
- Online Credential Stuffing Attack is a type of cyber attack where attackers use stolen username and password combinations (often obtained from data breaches) to gain unauthorized access to user accounts on various online platform.
- Offline Credential Stuffing Attacks on the other hand, deal with checking/correlating/corresponding already cracked `username/email:hash:password` pairs leaked from one srouce against another, once the latter has been cracked.
This way it's easy to identify vulnarable accounts that have been reusing passwords on multiple platforms.

### Why Not Crack Passwords in the Script?
While some offline credential stuffing attacks might use custom-built crackers in their scripts, this is inefficient and slow. Hashcat, on the other hand, is designed to efficiently crack hashes and is highly optimized for various hash functions. By using this Python script for correlation and leaving the cracking to Hashcat, you ensure that your credential stuffing workflow is much faster and more reliable.

### Usage
Run the following command to display the help menu:
```bash
$ python3 OCTA.py --help
usage: OCTA.py [-h] -b BASE [-m [MATCH ...]] [-d DIRECTORY] [-o OUTDIR]

Multi-source credential matcher

options:
  -h, --help            show this help message and exit
  -b BASE, --base BASE  Base credential file (username:hash:password)
  -m [MATCH ...], --match [MATCH ...]
                        One or more files to match against the base file
  -d DIRECTORY, --directory DIRECTORY
                        Directory containing multiple credential lists to match against the base file
  -o OUTDIR, --outdir OUTDIR
                        Output directory for match files (default: matches)
```

#### Example command:
```bash
python OCTA.py -b base -m list1
```

### Recommended Workflow
- Use Hashcat to try and crack as many hashes as you can from a list.
- Run this script to correlate the cracked `username/email:hash:password` pairs with another list containing usernames and unknown hashes.
- The output file will give you a list of matched credentials (`username:hash:password`) that can be used for further analysis.

## Features of this script
The script is essentially a multi-source credential matcher, because it not only supports checking credentials against one list, but multiple. It takes one basefile as an input `-b` and matches the basefile against the specified matchfiles. You can give matchfiles or lists to OCSTA with `-m` or `--match`. You are able to define as many as you like rigth after one another. Like so:

```bash
python3 OCSTA.py -b base -m list1 list2 list3
```

### Why do could you get mismatches?
When a mismatch occurs between hashes for the same username or email address, it can be attributed to several factors, including:
- Different Hashing Algorithms: The two systems may employ different hashing algorithms, such as Argon2id, bcrypt, or Scrypt. Each algorithm produces different hash outputs even for the same input, which leads to mismatches.
- Salting: Salting is a technique used to enhance security by adding a unique value (the salt) to each password before hashing it. Usually, all salts are randomly generated for each user, therefore, most likely, if salting is used, the resulting hashes will differ, even for identical passwords.
- Password Variants: Users may change their passwords across different platforms, resulting in different hashes for the same username or email address.
- Encoding Differences: Variations in how characters are encoded (e.g., UTF-8 vs. ASCII) can also lead to differing hashes for the same password. Although this isn't very common.

> [!important]
> It's important to note that if a hash mismatch occurs but the email address remains the same, it is more likely that the password will still match.
> This is because users that use email addressess as usernames use that email specifically, and chances are, if that same email is used on a different platform/service/server the password is likely to be the same.

### Folder Organization && Mismatches and comments
I paid attention to the organization of output files. By default a `matches` directory is created in the current working directory. Both the location of the folder and its name can be modified with `-o`. So, let's say you have a hash list from where you have already managed to crack 46% of the passwords. This list comes from a company named "Example Comany Corporations". You may also have gained access to list of usernames and hashes from "Steel Mountain INC" and you'd like to check if the usernames and hashes in your basefile corrolate with the already cracked ones or not.
If you run this command, inside the `test/matches` directory you will find all usernames that match with the usernames and hashes in your basefile. So there will be two files, because you inputted two lists to check: `ECC` and `Steel_Mountain-INC`.

```bash
python3 OCTA.py -b base -m ECC -m Steel_Mountain-INC -o test
```
Credentials that don't match at all are ignored. There will plenty of them. However, and this is more vital, there will be also credentials that the script detects as mismatches. Here is when that happens. The warning files will be created under the `test/warnings` directory under these condiditions:
    - A username from the basefile matches a username from a matchfile, but the hash associated with that username differs from what is found in the basefile. The script identifies at least one such mismatch and adds the entry in the format `username:hash:password:comment` to the respective warnings list.
- A username from the basefile matches a username from a matchfile, but the hash differs, and the username contains an "@" character (indicating that it is likely an email address). The script adds the entry to the warnings file and appends the comment: `Email found as username. Password may match.`

### How the output files are formatted
All outputs are formatted into markdown tables. The matchfiles have have `Usernames/Emails`, `Hashes`, `Passwords` and `Comments` as columns:
```bash
kali@kali:~/OCTA/matches/matches$ cat base_vs_list1_matches.txt
# Matches found in base vs list1

| Usernames/Emails  | Hashes                                                           | Passwords |
| ----------------- | ---------------------------------------------------------------- | --------- |
| user1             | cdb4ee2aea69cc6a83331bbe96dc2caa9a299d21329efb0336fc02a82e1839a8 | pass1     |
| user2             | 5ec1f7e700f37c3d0b2981d04855fc34b94aaa15457b05ca571817442d228f81 | pass2     |
| user3@example.com | 24d221b67b3728de7f769af4863d6bfaaa94b72bf41490697878ae96a61863c5 | pass3     |
```
And so do the warning files. They have have `Usernames/Emails`, `Hashes`, `Passwords` and `Comments`.
```bash
kali@kali:~/OCTA/matches/warnings$ cat base_vs_list1_warnings.txt
# Warnings for base vs list1

| Usernames/Emails  | Hashes                                                           | Passwords     | Comments                                     |
| ----------------- | ---------------------------------------------------------------- | ------------- | -------------------------------------------- |
| user1             | cdb4ee2aea69cc6a83331bbe96dc2caa9a299d21329efb0336fc02a82e1839a8 | Hash mismatch |                                              |
| user2             | 5ec1f7e700f37c3d0b2981d04855fc34b94aaa15457b05ca571817442d228f81 | Hash mismatch |                                              |
| user3@example.com | 1c3e5d6d3adddaa1a1e4dd33fa6fd2d0cdff2b3fc7712185f1e902cc6c20fa4b | Hash mismatch | Email found as username. Password may match. |
```
All these two above is filled with dummy data!


## Special directory mode
The special directory mode is enabled with `-d` or `--directory`. This will read every file from a user-defined directory and match them with the basefile.
In addition, simple statistics information is always displayed along with progress:
```bash
kali@kali:~/OCTA$ python3 OCTA.py -b basefile -d dir/
The directory 'matches' already exists. Overwrite? (y/n): y
Directory 'matches' has been cleared and will be recreated.
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 3/3 [00:00<00:00, 400.40it/s]

*** Processing Results ***
Files Processed        : 3
Failed Files           : 0
Total Matches found    : 6
Total Mismatches found : 4
```
