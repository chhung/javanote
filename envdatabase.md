# DataBase
## MySQL
### Docker setup
```shell
[docker@localhost ~]$ sudo docker pull mysql
[docker@localhost ~]$ sudo mkdir /home/docker/mysql-db
[docker@localhost ~]$ sudo docker run --name test-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-secret-pw -v /home/docker/mysql-db:/var/lib/mysql -d mysql:latest
```
### PL/SQL:建立資料庫和資料表
```sql
-- create database
CREATE DATABASE TestDB

-- 建立名為 Inventory 的新資料表
CREATE TABLE Inventory (SKU CHAR(5) NOT NULL, product VARCHAR(50), quantity INT, UNIQUE KEY (SKU))

-- 將資料插入新的資料表
INSERT INTO Inventory VALUES ('14256', 'banana', 150);
INSERT INTO Inventory VALUES ('24589', 'avocado', 7863783);
INSERT INTO Inventory VALUES ('35587', 'cherry', 34345345);
INSERT INTO Inventory VALUES ('55782', 'coconut', 179);
INSERT INTO Inventory VALUES ('77996', 'jujube', 14783);
INSERT INTO Inventory VALUES ('77837', 'durian', 47892);
INSERT INTO Inventory VALUES ('89653', 'grape', 258);
INSERT INTO Inventory VALUES ('98627', 'grapefruit', 47);
INSERT INTO Inventory VALUES ('87667', 'honeydew melon', 39387);
INSERT INTO Inventory VALUES ('64556', 'lemon', 39487);
INSERT INTO Inventory VALUES ('27287', 'orange', 387);
INSERT INTO Inventory VALUES ('78683', 'kiwi fruit', 337);
INSERT INTO Inventory VALUES ('12373', 'mangosteen', 797);
INSERT INTO Inventory VALUES ('36483', 'pear', 2786);
INSERT INTO Inventory VALUES ('25468', 'guava', 478);
```
### Draft
```
docker pull mysql/mysql-server:tag
docker run -it --restart=always --name test-mysql -p 3306:3306 -v /home/mkpl/mount/test-mysql:/var/lib/mysql -d mysql/mysql-server
docker run -it --restart=always --name test-mysql -v /home/mkpl/mount/test-mysql:/var/lib/mysql -d mysql/mysql-server

docker run -it --restart=always --name test-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-secret-pw -v /home/mkpl/mount/test-mysql:/var/lib/mysql -d mysql


docker logs test-mysql | grep GENERATED
docker exec -it test-mysql bash
bash-4.2# mysql -uroot -p
mysql> use mysql;
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'newegg.tw';
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'newegg.tw';
mysql> select host, user from mysql.user;
mysql> flush privileges;


新建一個使用者及權限(可從localhost以外登入):
drop user 'admin'@'localhost';
create user 'admin'@'%' identified by 'adminpassword';
grant all privileges on *.* to 'admin'@'%';
flush privileges;


重新設定密碼:
SET PASSWORD FOR 'user'@'localhost' = PASSWORD('password_txt');

UPDATE mysql.user SET Password = OLD_PASSWORD('newegg.tw') WHERE Host = 'localhost' AND User = 'root';

建立新資料庫:
mysql> CREATE DATABASE example_db;
mysql> show databases;
mysql> use example_db;

建立新資料表:
mysql> SHOW TABLES;
mysql> CREATE TABLE table_1 (name VARCHAR(30), birth_date DATE, sex CHAR(1), country VARCHAR(40));
mysql> describe table_1;

變更原有資料表的結構:
mysql> ALTER TABLE table_1 ADD email VARCHAR(50);
mysql> ALTER TABLE table_1 DROP birth_date;
```

---
## MSSQL
### Docker setup
```bash
[docker@localhost ~]$ sudo docker pull mcr.microsoft.com/mssql/server:2017-latest
[docker@localhost ~]$ sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourStrong@Passw0rd" -p 1433:1433 --name test-mssql -d mcr.microsoft.com/mssql/server:2017-latest
```
### T-SQL:建立資料庫和資料表
```sql
-- create database
CREATE DATABASE TestDB

-- 建立名為 Inventory 的新資料表
CREATE TABLE Inventory (SKU CHAR(5) PRIMARY KEY, name NVARCHAR(50), quantity INT)

-- 將資料插入新的資料表
INSERT INTO Inventory VALUES ('14256', 'banana', 150);
INSERT INTO Inventory VALUES ('24589', 'avocado', 7863783);
INSERT INTO Inventory VALUES ('35587', 'cherry', 34345345);
INSERT INTO Inventory VALUES ('55782', 'coconut', 179);
INSERT INTO Inventory VALUES ('77996', 'jujube', 14783);
INSERT INTO Inventory VALUES ('77837', 'durian', 47892);
INSERT INTO Inventory VALUES ('89653', 'grape', 258);
INSERT INTO Inventory VALUES ('98627', 'grapefruit', 47);
INSERT INTO Inventory VALUES ('87667', 'honeydew melon', 39387);
INSERT INTO Inventory VALUES ('64556', 'lemon', 39487);
INSERT INTO Inventory VALUES ('27287', 'orange', 387);
INSERT INTO Inventory VALUES ('78683', 'kiwi fruit', 337);
INSERT INTO Inventory VALUES ('12373', 'mangosteen', 797);
INSERT INTO Inventory VALUES ('36483', 'pear', 2786);
INSERT INTO Inventory VALUES ('25468', 'guava', 478);
```

---
## 建立關聯資料表
### 資料表生成
Student_Info![](https://i.imgur.com/j7KJJLs.png)

Department_Info![](https://i.imgur.com/b02rJuH.png)

Forgie Key為Student_Info的SCode，對映到Department_Info的DCode，我們希望如果系所的代號改變了，Student_Info的SCode欄位也要跟著改變。

Student_Info create code
```sql
CREATE TABLE `Student_Info` (
	`SNumber` INT(11) NOT NULL,
	`SName` VARCHAR(40) NOT NULL,
	`SCode` VARCHAR(10) NULL DEFAULT NULL,
	INDEX `Student_Info_ibfk_1` (`SCode`),
	CONSTRAINT `Student_Info_ibfk_1` FOREIGN KEY (`SCode`) REFERENCES `Department_Info` (`DCode`) ON UPDATE CASCADE ON DELETE SET NULL
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB;
```

以上的限制條件，也就是加入foreig key可以獨立寫
```sql
ALTER TABLE `Student_Info`
	ADD CONSTRAINT `Student_Info_ibfk_1` FOREIGN KEY (`SCode`) REFERENCES `Department_Info` (`DCode`) 
	ON DELETE SET NULL 
	ON UPDATE CASCADE;
```

Department_Info create code
```sql
CREATE TABLE `Department_Info` (
	`Did` INT(11) NOT NULL AUTO_INCREMENT,
	`DCode` VARCHAR(10) NULL DEFAULT NULL,
	`DAlias` VARCHAR(10) NOT NULL,
	`DTeacherName` VARCHAR(40) NOT NULL,
	PRIMARY KEY (`Did`),
	INDEX `DCode` (`DCode`)
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
AUTO_INCREMENT=3;
```