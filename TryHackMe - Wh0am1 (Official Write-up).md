# **TryHackMe - Wh0am1 (Official Write-up)**

## **Nmap** Scan:

Let's start with an nmap scan.

<u>**command:**</u> 

`nmap -sC -sV 192.168.252.142`

<u>**output:**</u>

![image-20210211122514573](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211122514573.png)

We see two ports open:

- *Port 22: OpenSSH* 
- *Port 80: HTTP*

## **Enumeration:**

**<u>Visiting the Webpage:</u>**

Upon visiting the web, we are presented with a GIF and some social media links. (Chromium browser opens up the web page more efficiently than Mozilla).

We view the page source, and find a comment:

![image-20210211123036860](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211123036860.png)

Probable a hidden directory. We visit the directory and we are presented with an Internal Portal, with a message from the 'administrator.'

![image-20210211123931031](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211123931031.png)

Since we don't have the credentials and the room instructions suggest that no brute-forcing is required, let's move towards another approach.

<u>**Directory Brute-forcing:**</u>

Let's fire up gobuster to enumerate some hidden directories and files.

<u>**command:**</u>
`gobuser dir -u http://192.168.100.65 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,sh,html`

**<u>output:</u>**

![image-20210211124741832](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211124741832.png)

And we find a bunch of directories, but the 'note.txt' file looks interesting. The note.txt file appears to be some sort of encrypted text. 

![image-20210211124816791](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211124816791.png)

Let's try base64 decoding the text as our first attempt to decryption. And voila we get a secret message from the admin to the development team.

**<u>command:</u>**
`echo -n VG8gdGhlIGRldiB0ZWFtLApNYWtlIHN1cmUgdG8gcmVtb3ZlIHRoZSBzZW5zaXRpdmUgZmlsZXMgZnJvbSBzZWNyZXRfbG9jYXRpb24uClRoYW5reW91IQotIGpvbiAoYWRtaW4p' | base64 -d`

**<u>output:</u>**

![image-20210211124911897](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211124911897.png)

Okay, so from the above message, we can infer three things:

1. *jon is the admin of the server*
2. *there's some sensitive file on the server that the development team has to remove*
3. *the 'secret_location' appears to be yet another directory, let's try to access it*

Surely, it is yet another directory, with an 'admin.txt' file.

![image-20210211125424973](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211125424973.png)

When we access the 'admin.txt' file, we are presented again with what appears to be a base64 encoded string.

![image-20210211125535396](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211125535396.png)

Let's go with base64 decoding again. And it's a response from the dev team to the admin.

**<u>command:</u>**
`echo -n 'VG8gam9uLApUaGFua3MgZm9yIHRoZSByZW1pbmRlci4gVGhlIHNhaWQgZmlsZXMgaGF2ZSBiZWVuIG1vdmVkIHRvIGEgYlc5eVpWOXpaV04xY21WayBsb2NhdGlvbiBmb3IgdGhlIHRpbWUgYmVpbmcuIFdlIHdpbGwgcmVtb3ZlIHRoZW0gY29tcGxldGVseSBvbmNlIHdlJ3JlIGRvbmUgdGhlIHNpdGUgZGV2ZWxvcG1lbnQuCi0gdGhlIGRldiB0ZWFtLg==' | base64 -d`

**<u>output:</u>**

![image-20210211125639807](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211125639807.png)

From this message, we can say that the sensitive file is still present on the server but has been moved to another location. When we try to access the location that the dev team has mentioned, we get a 404-Not Found error.

Since, all the conversation between the admin and the dev team until this point has been base64 encoded, it's a possibility that this location is also base64 encoded. Let's try to decode it. And once again, we find another directory, where we might find the mentioned sensitive file.

**<u>command:</u>**
`echo -n 'bW9yZV9zZWN1cmVk' | base64 -d`

**<u>output:</u>**

![image-20210211130142414](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211130142414.png)

We visit this directory and we get a creds.txt file which has the credentials of jon.

![image-20210211130638078](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211130638078.png)

When we access the Internal Portal with these creds, we get an authentication error. So, let's use these creds with SSH. And, we get logged in. Time to grab the user flag.

![image-20210211130816128](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211130816128.png)

Okay, so now for elevating our privileges, let's check for any sudo misconfigs.

**<u>command:</u>**
`sudo -l`

**<u>output:</u>**

![image-20210211130847199](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211130847199.png)

Alright! We can run 'apt' command with sudo.

Let's see how we can abuse this to escalate privileges. We visit https://gtfobins.github.io and search for apt. Looks like we can use elevate our privileges to root, using the command mentioned.

![image-20210211131120688](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211131120688.png)

After running this command, we have successfully achieved root access on this machine. Let's grab the root flag and answer the questions.

![image-20210211131342659](C:\Users\Shahrukh Iqbal Mirza\AppData\Roaming\Typora\typora-user-images\image-20210211131342659.png)



Question # 1: Who is the admin?
Answer: jon

Question # 2: In which directory is the sensitive file located? And what is the name of the file?
Answer: more_secured, creds.txt

Question # 3: What is the admin's password?
Answer: FlowersOfUnited06021958!!

Question # 4: What are the contents of the user.txt?
Answer: THM{<redacted>}

Question # 5: What are the contents of the root.txt?
Answer: THM{<redacted>}