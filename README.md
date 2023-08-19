# Murder-In-SQL-City
This challenge is from the LightHall Data Analytics Super League.
In this challenge, we were asked to aid our detective friend in identifying the culprit of a murder that occurred in SQL City

# MURDER IN SQL CITY!!

1. After importing the database, I decided to view the contents of each table to figure out where to start. The `crime_scene_report` table was an excellent starting point as it gave me insight as to when and where the murder took place and who were the witnesses to the crime. Since I vaguely remember the type of crime, the day, and the city it took place in, I typed the query below and the results it gave me are explained below it:

    ```
    SELECT date, type, description, city 
    FROM crime_scene_report
    WHERE date = "20180115" AND city = "SQL City" AND type = "murder";
    ```

   The above code snippet produced the insight below:

   According to the `description` column from the `crime_scene_report` table, security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

2. The next step was to determine the details of these two witnesses or at the very least, their names.

    ```
    SELECT * FROM person
    WHERE address_street_name = "Franklin Ave" OR address_street_name = "Northwestern Dr"
    ORDER BY address_number ASC;
    ```

    The above query determined that the first witness is called Morty Schapiro and the second witness is called Annabel Miller. Morty Schapiro has an ID number of “14887” and Annabel Miller has an ID number of “16371”.

3. Evidence through interviews with the witnesses point towards the fact that the perpetrator was a gold member of the “Get-fit-now gym. See the query below:

    ```
    SELECT * FROM interview
    WHERE person_id = "14887" OR person_id = "16371";
    ```

    Morty Schapiro had this to say: "I heard a gunshot and then saw a man run out. He had a 'Get Fit Now Gym' bag. The membership number on the bag started with '48Z'. Only gold members have those bags. The man got into a car with a plate that included 'H42W'."

    Annabel Miller had this to say: “I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January 9th.”

4. Based on these two witness reports, I headed to the `get_fit_now` and `get_fit_now_check_in` tables to close in on the culprit.

    Using the query below, I was able to narrow it down to two suspects who went to the gym on January 9th, 2018, and have their gym memberships starting with “48Z”:

    ```
    SELECT * FROM get_fit_now_check_in
    WHERE check_in_date = "20180109" AND membership_id LIKE "48Z%";
    ```

    These two members are 48Z7A & 48Z55.

5. By joining the `person` table and the `get_fit_now_member` tables, I was able to determine the details of the two members with their membership IDs being 48Z7A & 48Z55. The query below shows the process:

    ```
    SELECT * FROM person
    INNER JOIN get_fit_now_member 
    ON get_fit_now_member.person_id = person.id
    WHERE get_fit_now_member.id = "48Z55" OR get_fit_now_member.id = "48Z7A";
    ```

    They are “Jeremy Bowers” and “Joe Germuska”. Jeremy’s details are as follows:
    - License ID: 423327
    - Address number: 530
    - Address street name: Washington Pl, Apt 3A
    - SSN: 871539279
    Joe’s details are as follows:
    - License ID: 173289
    - Address number: 111
    - Address street name: Fisk Rd
    - SSN: 138909730

6. Now, according to Morty, our prime suspect is a male who got into a car with a license plate including these characters “H42W”. Using the query below, it was determined that there are three plates that contain the key characters identified by the witness:

    ```SQL
    SELECT * FROM drivers_license
    WHERE plate_number LIKE "%H42W%";
    ```

    The details of the three are:
    - A Toyota Prius with plate number “H42W0X”, belonging to a female with blue eyes, blonde hair, age 21, and ID number 183779.
    - A Chevrolet Spark LS with plate number “0H42W2”, belonging to a male with brown eyes, brown hair, age 30, and ID number 423327.
    - A Nissan Altima with plate number “4H42WR”, belonging to a male with black eyes, black hair, age 21, and ID number 664760.
   
   However, we can rule out the female since it was a male seen coming out of the gym. This leaves us with the owners of the Chevrolet Spark and the Nissan Altima.

7. By joining the `person` table to the `drivers_license` table, I was able to determine the owners of these two vehicles. The code snippet below shows that:

    ```
    SELECT * FROM person
    JOIN drivers_license
    ON person.license_id = drivers_license.id
    WHERE person.license_id = "423327" OR person.license_id = "664760";
    ```

    As expected, one vehicle belongs to one of the two suspects identified earlier, Jeremy Bowers, while the other belongs to Tushar Chandra.

8. Upon seeing this new name, I quickly went to verify if this "Tushar Chandra" was a member of the Get-Fit-Now gym by using the code snippet below, and the result set was null! That means the name is not in the list:

    ```
    SELECT * FROM get_fit_now_member
    WHERE name = "Tushar Chandra";
    ```

    This can only mean one thing. However, let’s recap a bit:
    - From the witness interviews, it was concluded that the suspect was male, a gold member of the Get-Fit-Now gym, and was seen escaping in a vehicle with a license plate that contains “H42W”.
    - Next, it was discovered that there were two males in the gold gym membership class that fit the description, namely, Jeremy Bowers and Joe Germuska.
    - Moving on, I was able to narrow down the vehicles which contain the key characters we were looking for.
    - Afterwards, I used the ID number used to register said vehicles and then came up with two names, Jeremy Bowers and Tushar Chandra. Jeremy is not new to us, but Tushar is. So I went back to the gym members' table and crosschecked to see if I could find a Tushar Chandra in the list. However, Tushar does not go to that gym.

9. In conclusion, the prime suspect that was identified by the witnesses as a male who is a gold member at the Get-Fit-Now gym and was seen fleeing the scene in a vehicle with a license plate containing characters “H42W” was none other than JEREMY BOWERS. He is the owner of a Chevrolet Spark LS with license plate “4H42WR”. He is a gold member of the Get-Fit-Now gym with membership ID “48Z55”. He lives on Washington Pl, Apt 3A, and has a Social Security Number of 871539279.

10. To confirm this, I used the query below:

    ```
    INSERT INTO solution (user, value)
    VALUES ("1", "Jeremy Bowers");
    ```

    And then queried the solution:

    ```
    SELECT * FROM solution;
    ```

    The result confirmed Jeremy Bowers as the murderer. However, there's more to this case than expected! It seems he was hired to carry it out; there's a real villain out there that needs to be caught!

11. Using the hint obtained earlier, I headed to the interview table to see what the murderer had to say by using the query below:

    ```
    SELECT * FROM interview
    WHERE person_id = "67318";
    ```

    This is what he had to say:
    “I was hired by a woman with a lot of money. I don't know her name, but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.”

12. Following the confession of the murderer, I went to the `facebook_event_checkin` table to determine the details of the SQL Symphony Concert using the query below:

    ```
    SELECT * FROM facebook_event_checkin
    WHERE event_name = "SQL Symphony Concert";
    ```

    It was then determined that the event ID for this event was “1143”.

13. By adding to the query, I then narrowed it down further to identify how many people attended this event 3 times in December 2017:

    ```
    SELECT COUNT(*), person_id, event_name
    FROM facebook_event_checkin
    WHERE event_id = "1143" AND date LIKE "201712%"
    GROUP BY person_id
    HAVING COUNT(*) > 2;
    ```

    This gave me the IDs of two people: 24556 & 99716.

14. Obtaining their IDs, I moved on to check the `person` table to find out more details about these two new suspects by using the query below:

    ```
    SELECT * FROM person
    WHERE id = "24556" OR id = "99716";
    ```

    The insights gained from this indicate that:
    - ID number “24556” belongs to Bryan Pardo who lives at 703, Machine Ln with a License ID of 101191 and Social Security Number 816663882.
    - ID number “99716” belongs to Miranda Priestly who lives at 1883, Golden Ave with a License ID of 202298 and Social Security Number 987756388.

    However, from the murderer’s confession, he indicated that it was a woman that hired him, which means Bryan is out of the question leaving Miranda to be investigated further.

15. After ruling out Bryan, I now doubled down on our last remaining suspect, Miranda Priestly. Using the license ID I got from the previous query, I then decided to cross-check the details of that ID in the `drivers_license` table:

    ```
    SELECT * FROM drivers_license
    WHERE id = "202298";
    ```

    This query confirmed the murderer’s confession as the owner of this license, Miranda Priestly, fits the description he gave:
    - Height: 66”
    - Age: 68
    - Eye color: green
    - Hair color: red
    - Gender: Female
    - Plate number: 500123
    - Car: Tesla Model S.

16. Based on these descriptions of the one who hired the murderer, we can conclude that the real villain is MIRANDA PRIESTLY! She’s the one who hired Jeremy Bowers to carry out the murder.


   


