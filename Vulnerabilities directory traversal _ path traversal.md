# Vulnerabilities directory traversal / path traversal
**1. Directory Structure in Unix-based Operating Systems** 
Hierarchical Structure: The directory structure is organized in a hierarchical tree-like format.
![Screenshot 2024-05-31 140618](https://hackmd.io/_uploads/B1kGwgv4C.png)
1. File Paths: A file path starts from the root directory and moves through parent directories, subdirectories, until reaching the target file. Each level is separated by a slash `(/)`.
Ex: 
![Screenshot 2024-05-29 163726](https://hackmd.io/_uploads/HJORDgDN0.png)

 
2. ./ (Current Directory): Indicates the current directory. Though it seems harmless, it can sometimes bypass filters in directory traversal vulnerabilities.
EX:
```
ls /root/carvingLab/./
ls /root/././carvingLab
```
    
    
3. ../ (Parent Directory) 
Used to navigate to the parent directory: 
![Screenshot 2024-05-29 164650](https://hackmd.io/_uploads/SJxStlP4R.png)


Moving between directories: 
![Screenshot 2024-05-29 165040](https://hackmd.io/_uploads/HJOKtgPVC.png)




**2. Target Files in Directory Traversal Vulnerabilities**
* /etc/passwd

![Screenshot 2024-05-30 132533](https://hackmd.io/_uploads/HkaDqxDNR.png)
 Contains user information such as usernames, passwords (denoted by 'x'), user IDs, group IDs, GECOS, home directory, and shell.

![Screenshot 2024-05-31 142210](https://hackmd.io/_uploads/HJXTqeDVR.png)


*  /etc/apache2/*
Directory containing Apache configuration files, which can reveal server information like web index and server ports.

![Screenshot 2024-05-30 181626](https://hackmd.io/_uploads/rkOwilw4R.png)


*  /etc/nginx/
 Similar to /etc/apache2/, contains Nginx configuration files with web directory and port information.

*  /etc/environment
Contains environment variables that may include sensitive information like secret keys.

**3. Analyzing and Exploiting Vulnerabilities**
1. The vulnerability occurs when using functions that read files and trust user input
EX: 
```
<?php

if (isset($_GET['file'])) {
    $file = $_GET['file'];
    include($file);
}

?>
```
Code explanation: the user will upload a file using the GET method and then the system uses the `include()` function to display the content of that `$file`
It can be easily seen that the above code does not have any filter for `$file` . This is a sign of a Directory traversal vulnerability 


EX: https://portswigger.net/web-security/file-path-traversal/lab-simple
Here is a lab on portSwigger with the challenge description as *“This lab contains a path traversal vulnerability in the display of product images.
To solve the lab, retrieve the contents of the `/etc/passwd` file.”*
When we go to the source of this website, we can see the lines
```
 <img src="/image?filename=5.jpg">
 <h3>Cheshire Cat Grin</h3>
  <img src="/resources/images/rating2.png">
```
có thể thấy ảnh được tải từ thư mục `/image`
![Screenshot 2024-05-31 085106](https://hackmd.io/_uploads/SyQJybvN0.png)


Go Burp Suite

![Screenshot 2024-05-31 085552](https://hackmd.io/_uploads/S1mbkWPNR.png)

From here you can see the `filename` parameter can be changed + challenge description -> use Directory traversal vulnerability to go to `/etc/passwd` directory
![Screenshot 2024-05-31 085807](https://hackmd.io/_uploads/SkFLJbPVA.png)


yaaaa, here I don't know how many steps it will have to go back to reach the root directory so I just go back slowly and 🎉
![Screenshot 2024-05-31 085936](https://hackmd.io/_uploads/SyCvyZv4R.png)

Here is right path: 
**/image?filename=../../../etc/passwd**






2. Bypass filter removes directly ../
```
<?php

if (isset($_GET['file'])) {
    $file = $_GET['file'];
    $search = "../";
    $replace = "";
    $file_removed_traversal = str_replace($search, $replace, $file);
    include($file_removed_traversal);
    exit();
}

?>
```
Basically, this PHP snippet will take the file from the user and find if the first character of the file is `/` not to prevent absolute paths, then search `../` to replace `../` with the space `""` . The str_replace function will only execute once, for example: ab`../`c will become abc. At this time, it will still have the path traversal vulnerability as usual because for example, if I put `....//`, str_replace will only be removed once and `../` is the result after filtering.
EX:https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively

Discripe challenge: 
![Screenshot 2024-05-31 095809](https://hackmd.io/_uploads/HJRKxbv4C.png)


It's still the old web interface, now we'll go straight to Burp Suite

![Screenshot 2024-05-31 095908](https://hackmd.io/_uploads/H1U3eWvV0.png)

At this point I will try changing the folder path to see if anything happens
![Screenshot 2024-05-31 095921](https://hackmd.io/_uploads/Skn0gWw4R.png)

woooo, it's still the same, maybe it's been removed `../` but I'll try another request and see `....//` 


At this time the response is “No such file”. This is actually a path traversal vulnerability
I will now use it to go to the `etc/passwd` folder as shown in the challenge description
![Screenshot 2024-05-31 095958](https://hackmd.io/_uploads/BymxfZwNA.png)

Because I don't know how much I need to go back to reach the Root folder, I will go back slowly by adding more times ....// 
![Screenshot 2024-05-31 100243](https://hackmd.io/_uploads/SkLXf-D4C.png)

và đây là đường dẩn đúng: **/image?filename=....//....//....//etc/passwd**


3. File path traversal, traversal sequences stripped with superfluous URL- decode

In this vulnerability, the browser will filter `/` characters. To bypass this vulnerability we can use URL encoding as` / `to `%2fetc%2fpasswd`

But somtime the URL will decode our request, then `%2fetc%2fpasswd` will become /ect/passwd now the web filter will still filter / so we need to decode `%2f` one more time: `%2f -> %252f`

Continuing with a lab challenge about path traversal on portSwigger
EX:https://portswigger.net/web-security/file-path-traversal/lab-superfluous-url-decode

![Screenshot 2024-05-31 123542](https://hackmd.io/_uploads/ryTxX-w4C.png)

First, use the encoded path once to see how it looks
![Screenshot 2024-05-31 125857](https://hackmd.io/_uploads/SyvB7ZvNC.png)

Next we will attack normally
![Screenshot 2024-05-31 125956](https://hackmd.io/_uploads/B1APXWwE0.png)



hmmm still nothing, maybe it was in case the URL decoded our payload once to `/` and the web filtered it
So now I will encode a second time
![Screenshot 2024-05-31 130102](https://hackmd.io/_uploads/HJaiQZwVR.png)

well, that idea was correct. this is actually a path traversal vulnerability.

 4. Bypass vulnerability when website uses full path
```
<?php

if (isset($_GET['file'])) {
    $file = $_GET['file'];
    include('/home/Phuong/Desktop/DiretoryTraversal/' . $file);
}

?>

$file khi người dùng tải lên sẽ được lưu ở thư mục /home/Viblo/Desktop/DiretoryTraversal/ 
  /home/Phuong/Desktop/DiretoryTraversal/../../../../../../../../etc/passwd
= /home/Phuong/Desktop/../../../../../../../etc/passwd
= /home/Phuong/../../../../../../etc/passwd
= /home/../../../../../etc/passwd
= /../../../../etc/passwd
= /etc/passwd
```

Like here, I will go back to root and then from root to /ect/passwd

Can you try exploiting through this portSwigger lab: https://portswigger.net/web-security/file-path-traversal/lab-validate-start-of-path

 
**5. Useful tools** 

Burp Suite: Can be used to detect and exploit Path Traversal vulnerabilities through modifying HTTP requests and examining server responses.
![Screenshot 2024-05-31 125807](https://hackmd.io/_uploads/B1PMBWPVC.png)





WFuzz: Performs fuzzing on URL parameters to detect Path Traversal vulnerabilities.

![th](https://hackmd.io/_uploads/B1M6SWP4A.jpg)




Dirsearch: Dirsearch is an open source tool written in Python, designed to brute force directories and files on web servers. 

![Screenshot 2024-05-31 150737](https://hackmd.io/_uploads/Hk3Rrbv4C.png)


6. Prevention:
You should validate user input before processing it: use dirname`($filePath)` to get the path of that file.


Use whitelist for allowed values.


Or the file name is alphanumeric characters and should not contain special characters.
