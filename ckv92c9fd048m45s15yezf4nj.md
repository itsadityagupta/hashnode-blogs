---
title: "MongoDB Setup"
datePublished: Wed Oct 27 2021 05:14:21 GMT+0000 (Coordinated Universal Time)
cuid: ckv92c9fd048m45s15yezf4nj
slug: mongodb-setup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1635311565406/3e21CWpomO.png
tags: nosql, mongodb, databases, setup

---

Follow the below instructions to set up a MongoDB server in your system.

1. Go to https://www.mongodb.com/try/download/community
2. Download the MongoDB Community Server as per your system's OS.
3. Install the MongoDB Community Server using the downloaded setup using the default configurations.
4. It should have been installed in the path: `C:\Program Files\MongoDB`
5. Now, visit https://www.mongodb.com/try/download/shell
6. Download the MongoDB Shell.
7. Install it. I must be installed in the path: `C:\Users\<username>\AppData\Local\Programs\mongosh`
8. In the above path, there are 2 files: `mongosh.exe` and `mongocryptd-mongosh.exe`. Copy these 2 inside `C:\Program Files\MongoDB\Server\5.0\bin`
9. Open command prompt as an administrator.
10. To make sure that the MongoDB server is running, enter the command `net start MongoDB`. It'll start the MongoDB server, if not running already.
11. Then enter the command `mongosh`. A shell to open MongoDB commands will open.

If the `mongosh` command is successful, you've set up MongoDB in your system successfully.