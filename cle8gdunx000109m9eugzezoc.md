---
title: "Linkedin Database Design"
datePublished: Fri Feb 17 2023 11:34:08 GMT+0000 (Coordinated Universal Time)
cuid: cle8gdunx000109m9eugzezoc
slug: linkedin-database-design
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1676633094986/be3a9a7a-d4e4-4b4a-b8b9-c74cb56b8ea9.png
tags: sql, data-modeling, dataengineering

---

## Introduction

LinkedIn stands out as a professional networking platform with a user base of over 750 million professionals worldwide. It handles an enormous amount of user data, including profiles, connections, job postings, and much more. In this blog post, I'll design a database for Linkedin and understand the various entities and relationships between them.

To keep the scope of this blog limited, here are the requirements that we want our database to satisfy:

1. Users can **create their profile** with information like name, contact information, headline, summary, education, work experience and different skills.
    
2. Users can **send connection requests** to other users, or **follow them** only. They should also be able to **accept or reject** the connection request.
    
3. Users can **create posts**, and **like** and **comment** on others' posts.
    

> You can view the complete SQL code to create the below design along with the ER diagram on my github repo [here](https://github.com/Aditya-Gupta1/data-engineering-projects/tree/linkedin-db-design-oltp).

## User Profile

Each user will have some basic information like name, current location, headline and summary. Along with this, each user can have multiple work experiences and multiple education details. Considering these things, we can have the following tables:

* **UserProfiles**:  
    ▸ user\_id (Primary Key)  
    ▸ first\_name  
    ▸ middle\_name  
    ▸ last\_name  
    ▸ contact\_information\_id (Foreign Key to the *ContactInformation* table)  
    ▸ headline  
    ▸ location  
    ▸ about\_section
    
* **ContactInformation**:  
    ▸ contact\_information\_id (Primary Key)  
    ▸ email  
    ▸ email\_visibility  
    ▸ phone\_no  
    ▸ phone\_no\_visibility  
    ▸ website  
    ▸ website\_visibility  
    Visibility can be one of *"PUBLIC", "PRIVATE" or "CONNECTIONS\_ONLY".*
    
* **Skills**:  
    ▸ skill\_id (Primary Key)  
    ▸ skill\_name
    
* **EducationDetails**:  
    ▸ education\_id (Primary Key)  
    ▸ school\_id (Foreign Key to the *Schools* table)  
    ▸ degree  
    ▸ field\_of\_study  
    ▸ start\_date  
    ▸ end\_date  
    ▸ grade  
    ▸ description
    
* **Schools**:  
    ▸ school\_id (Primary Key)  
    ▸ school\_name  
    ▸ location  
    ▸ website
    
* **UserEducationDetails**:  
    ▸ user\_id (Primary Key, Foreign Key to the *UserProfiles* table)  
    ▸ education\_id (Primary Key, Foreign Key to the *EducationDetails* table)  
    This table records the user and their corresponding education details.
    
* **EducationSkills**:  
    ▸ education\_id (Primary Key, Foreign Key to the *EducationDetails* table)  
    ▸ skill\_id (Primary Key, Foreign Key to the *Skills* table)  
    Associates skills with education.
    
* **UserSkills**:  
    ▸ user\_id (Primary Key, Foreign Key to the *UserProfiles* table)  
    ▸ skill\_id (Primary Key, Foreign Key to the *Skills* table)  
    Associate users with the skills.
    
* **Companies**:  
    ▸ company\_id (Primary Key)  
    ▸ company\_name  
    ▸ company\_website  
    ▸ about\_company
    
* **Experiences**:  
    ▸ experience\_id (Primary Key)  
    ▸ company\_id (Foreign Key to the *Companies* table)  
    ▸ profile\_headline  
    ▸ employment\_type  
    ▸ start\_date  
    ▸ end\_date  
    ▸ location\_type  
    ▸ employment\_location  
    ▸ is\_current\_role  
    ▸ employment\_industry  
    ▸ description  
    Here, `employment_type` can be one of "*FULL-TIME", "PART-TIME", "SELF-EMPLOYED", "FREELANCE", "INTERNSHIP" or "TRAINEE",* and `location_type` can be one of *"ON-SITE", "HYBRID" or "REMOTE"*.
    
* **ExperienceSkills**:  
    ▸ experience\_id (Primary Key, Foreign Key to the *Experiences* table)  
    ▸ skill\_id (Primary Key, Foreign Key to the *Skills* table)  
    Associates experience with skills.
    
* **UserExperience**:  
    ▸ user\_id (Primary Key, Foreign Key to the *UserProfiles* table)  
    ▸ experience\_id (Primary Key, Foreign Key to the *Experiences* table)  
    Associates users with experiences.
    

This completes a user's profile.

## Connection Requests

We need to record who sent a connection request to whom, along with its status, and the followers of a user. Here are the tables to record this:

* **Connections**:  
    ▸ connection\_id (Primary Key)  
    ▸ request\_sent\_by (Foreign Key to the *UserProfiles* table)  
    ▸ request\_sent\_to (Foreign Key to the *UserProfiles* table)  
    ▸ request\_status  
    `request_status` can be one of *"CONNECTED" or "PENDING"*.
    
* **Followers**:  
    ▸ followed\_by (Primary Key, Foreign Key to the *UserProfiles* table)  
    ▸ following (Primary Key, Foreign Key to the *UserProfiles* table)
    

These 2 tables can record information for connections and followers.

## Posts and Comments

A user can create a post, and like and comment on others' posts. We can have the following tables for this:

* **Posts**:  
    ▸ post\_id (Primary Key)  
    ▸ user\_id (Foreign Key to the *UserProfiles* table)  
    ▸ description  
    ▸ created\_at  
    ▸ updated\_at
    
* **PostReactions**:  
    ▸ post\_id (Primary Key, Foreign Key to the *Posts* table)  
    ▸ user\_id (Primary Key, Foreign Key to the *UserProfiles* table)  
    ▸ reaction  
    `reaction` can be one of *'"LIKE'", "CELEBRATE", "SUPPORT", "FUNNY", "LOVE", or "INSIGHTFUL"*.
    
* **Comments**:  
    ▸ comments\_id (Primary Key)  
    ▸ post\_id (Foreign Key to the *Posts* table)  
    ▸ user\_id (Foreign Key to the *UserProfiles* table)  
    ▸ description  
    ▸ created\_at  
    ▸ updated\_at
    
* **CommentReactions**:  
    ▸ comments\_id (Primary Key, Foreign Key to the *Comments* table)  
    ▸ user\_id (Primary Key, Foreign Key to the *UserProfiles* table)  
    ▸ reaction
    

## Complete Design

Here's a complete database design for the tables mentioned above:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676631576153/3c4c27ce-fa6a-486c-9cb0-7f5e232b93bf.png align="center")

## Conclusion

Overall, there are **18 tables** in this design. This design represents only the basic functionality. A lot more features can be added to this such as:

* LinkedIn Groups
    
* Linkedin Pages
    
* Linkedin Articles
    
* Block a User
    
* Featured Section in a user's profile
    

and many more.

Also, this design is for the **OLTP database**. For analytical processing or warehousing, we would need a much more denormalized design. But that's a topic for another blog.

Thanks for reading!