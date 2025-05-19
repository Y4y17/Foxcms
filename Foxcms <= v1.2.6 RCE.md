# Foxcms v1.2.6 Arbitrary File Upload Vulnerability
##### Vulnerability Location: /app/admin/controller/PicManager.php
##### Affected Range: <= Foxcms v1.2.6
##### Vulnerability Cause: PicManager.php contains a serious security vulnerability that allows an attacker to upload malicious template files, resulting in remote code execution vulnerabilities.
##### Vulnerability Impact: Full Server Takeover
##### Link: https://www.foxcms.cn/
# Vulnerability recurrence
Download the source code, using phpstudy structures, environment, the interface is as follows:

<img width="1512" alt="image" src="https://github.com/user-attachments/assets/b2efec73-5103-48da-93f7-713cc0f23434" />

Log in to the background to grab data packets and obtain Cookie information. Local host starts http service:

![image](https://github.com/user-attachments/assets/4057bc80-d3cc-4a08-8b03-d9216f02b1c0)

Look for the functional points for image upload as follows:

<img width="1489" alt="image" src="https://github.com/user-attachments/assets/9606cd57-8003-4b82-be6c-afecc72e5431" />

Select network extraction:

<img width="967" alt="image" src="https://github.com/user-attachments/assets/24ab5164-3c36-42f0-8cc7-3eb03dbd5fc8" />

There is a malicious php file in the local http service directory. The test.php file exists below, and its content is as follows:

![image](https://github.com/user-attachments/assets/bc1e9105-93f3-409e-9d61-6052ed2a130d)


Enter any image address at random, such as: https://www.baidu.com/img/flexible/logo/pc/result@2.png Capture data packets, modify the remote image address to the address of the malicious script:

<img width="1316" alt="image" src="https://github.com/user-attachments/assets/d144ce2e-1c2c-43bb-b2f1-1caf41955b17" />

Although the status code of the echo packet is 500, it can be found that the malicious file has been downloaded and saved on the server!

![image](https://github.com/user-attachments/assets/4f753e73-7d4b-44cf-bd49-e3293ae80897)

Try to access a malicious files: http://x.x.x.x/uploads/20250519/test.php

<img width="1346" alt="image" src="https://github.com/user-attachments/assets/d0e53ddc-ada4-4526-b87e-4412e2dfc850" />

# Code Analysis

The picManager function exists in the picmanager.php file. When the variable type=extract, the getImg method will be called:

![image](https://github.com/user-attachments/assets/5e5ea212-3358-4723-aae8-b07c705a889a)

Following up on the getImg method, it was found that the download method would be called to download remote images

![image](https://github.com/user-attachments/assets/d572db44-cea0-4629-9094-b4bf44eb1d44)

In the download method, the file will be directly written to the server through the file_put_contents function!

![image](https://github.com/user-attachments/assets/664ba665-36bb-4081-aa00-d58db97b740f)

Back in the getImg method, the upload method will be called afterwards, and then follow up in this method:

![image](https://github.com/user-attachments/assets/88a2c5d0-ea1b-49b3-89b6-7e764565a444)

The original type of the obtained file will be obtained above! Then the validationSuffix function determines whether the file type is valid!

![image](https://github.com/user-attachments/assets/5f2488c3-911a-4f22-b6af-be3e5dc3bc0d)

There is an upload whitelist：

![image](https://github.com/user-attachments/assets/e345a448-fe45-4566-8d96-a9bc2bffd113)

There is no satisfaction here! So it returned "code=0", and eventually prompted that the upload failed, as well as subsequent status codes and other information. There is a logical error here: The type was determined after the file was written to the server!
