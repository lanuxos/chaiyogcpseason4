# Google Cloud Computing Foundations: Infrastructure in Google Cloud - Locales

## where do i store the staff?
- connect to database
gcloud sql connect myinstance --user=root
- create database
CREATE DATABASE guestbook;
- insert table, records
```
USE guestbook;
CREATE TABLE entries (guestName VARCHAR(255), content VARCHAR(255),
    entryID INT NOT NULL AUTO_INCREMENT, PRIMARY KEY(entryID));
    INSERT INTO entries (guestName, content) values ("first guest", "I got here!");
INSERT INTO entries (guestName, content) values ("second guest", "Me too!");
```
- retrieve data
SELECT * FROM entries;