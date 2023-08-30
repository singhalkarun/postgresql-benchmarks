Environment (2 Core CPU, 8 GB Ram)

Tables:

Student
Teacher
School
Assessment

1. Create table teacher, school and student with (id, name)

1. Insert Teachers <- Took 1 second 15 ms

```sql
INSERT INTO teacher (id, name) VALUES (generate_series(1, 200000), 'default')
```

2. Insert Students <- Took 58 secs 227 ms

```sql
INSERT INTO student (id, name) VALUES (generate_series(1, 10000000), 'default')
```

3. Insert Schools <- Took 485 ms

```sql
INSERT INTO school (id, name) VALUES (generate_series(1, 50000), 'default')
```

4. Create hypertable assessment

```sql
CREATE TABLE assessment (
	id serial,
	submission_time timestamptz,
	school_id int,
	teacher_id int,
	student_id int,
	grade int,
	is_passed bool,
	FOREIGN KEY (school_id) REFERENCES school(id),
	FOREIGN KEY (teacher_id) REFERENCES teacher(id),
	FOREIGN KEY (student_id) REFERENCES student(id)
)

SELECT create_hypertable('assessment', 'submission_time')
```
5. Insert 10000 randomised assessment <- Took 1 secs 282 msec

```sql
INSERT INTO assessment 
(id, submission_time, school_id, student_id, teacher_id, grade, is_passed) 
VALUES 
(generate_series(1, 10000), timestamptz '2023-07-01 00:00:00 UTC' + random() * INTERVAL '30 days', floor(random() * 50000) + 1, floor(random() * 10000000) + 1, floor(random() * 200000) + 1, floor(random() * 3) + 1, RANDOM()::INT::BOOLEAN);
```

6. Insert 90000 randomised assessment <- Took 17 secs

```sql
INSERT INTO assessment 
(id, submission_time, school_id, student_id, teacher_id, grade, is_passed) 
VALUES 
(generate_series(10001, 100000), timestamptz '2023-07-01 00:00:00 UTC' + random() * INTERVAL '30 days', floor(random() * 50000) + 1, floor(random() * 10000000) + 1, floor(random() * 200000) + 1, floor(random() * 3) + 1, RANDOM()::INT::BOOLEAN);
```

7. Insert 100000 randomised assessment <- Took 10 secs

```sql
INSERT INTO assessment 
(id, submission_time, school_id, student_id, teacher_id, grade, is_passed) 
VALUES 
(generate_series(100001, 200000), timestamptz '2023-07-01 00:00:00 UTC' + random() * INTERVAL '30 days', floor(random() * 50000) + 1, floor(random() * 10000000) + 1, floor(random() * 200000) + 1, floor(random() * 3) + 1, RANDOM()::INT::BOOLEAN);
```

8. Insert 8,00,000 randomised assessment <- Took 1 min 8 Secs

```sql
INSERT INTO assessment 
(id, submission_time, school_id, student_id, teacher_id, grade, is_passed) 
VALUES 
(generate_series(200001, 1000000), timestamptz '2023-07-01 00:00:00 UTC' + random() * INTERVAL '30 days', floor(random() * 50000) + 1, floor(random() * 10000000) + 1, floor(random() * 200000) + 1, floor(random() * 3) + 1, RANDOM()::INT::BOOLEAN);
```

7. Query Count of all assessment for a particular week

```sql
explain analyze select count(*) from assessment where submission_time between '2023-07-03' and '2023-07-09';
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/0c8fe693-e6aa-4c56-bb27-34787bbf5db5)

8. Insert 10,00,000 randomised assessment <- Took 1 min 28 Sec
9. Insert 20,00,000 randomised assessment <- Took 2 min 56 Sec
10. Insert 20,00,000 randomised assessment <- Took 2 min 56 Sec
10. Insert 40,00,000 randomised assessment <- Took ~ 6 min

11. Query Count of all assessment for a particular week

```sql
explain analyze select count(*) from assessment where submission_time between '2023-07-03' and '2023-07-09';
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/27e22ba0-a530-4b95-8524-839af9e41635)

12. Chunks created by timescaledb

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/777e8062-f321-4a93-ac70-fb4b673386cb)

13. Query a non-hypertable with exactly same data

```sql
explain analyze select count(*) from assessment_non_hypertable where submission_time between '2023-07-03' and '2023-07-09';
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/ddf54fed-b4be-410d-be87-b854a9b3d9d0)

14. Insert 10000000 assessment
15. Query Count of all assessment for a particular week

```sql
explain analyze select count(*) from assessment where submission_time between '2023-07-03' and '2023-07-09';
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/339e88ac-26df-4660-b09a-44e8a495cb66)

16.Query Count of all assessment for a particular week in a non-hypertable with exact same data
![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/4f9327f2-8e23-4c8d-8ca9-9f1f50935cbf)

17. Insert 1,00,00,000 assessment in different month
18. Query Count of all assessment for a particular week

```sql
explain analyze select count(*) from assessment where submission_time between '2023-07-03' and '2023-07-09';
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/546f0a1d-0136-455f-9ccd-e02ed79cfb09)

19. Query Count for non-hypertable

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/97ad8d2a-5519-491e-94b0-3c78ab4aa83d)

20. Insert 1,00,00,000 assessment in different month
21. Query count of all assessment for a particular week

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/b38b45b6-d68e-4a24-b99a-3dd9732c6e9f)

22. Same query for non-hypertable

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/5bfa728f-f838-4557-b716-f61851973c99)

23. Let's create index for non-hypertable

```sql
CREATE INDEX assessment_non_hypertable_submission_idx ON assessment_non_hypertable (submission_time)
```
