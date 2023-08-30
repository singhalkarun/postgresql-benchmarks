## Benchmarking Query Times

### VM Environment

- Azure VM - Standard B2ms (2 vcpus, 8 GiB memory)
- Disk - Premium SSD LRS - 30 GB

### Compose File to Setup Database

```yml
version: '3.8'
services:
  postgresql:
    image: postgres:15-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - '5432:5432'
    volumes:
      - /data/postgresql:/var/lib/postgresql/data
```

```sql
CREATE TABLE public.school (
id serial,
PRIMARY KEY (id),
name varchar(255)
);

CREATE TABLE public.student (
id serial,
name varchar(255),
school_id int,
PRIMARY KEY (id),
FOREIGN KEY (school_id) REFERENCES  school(id)
);

CREATE TABLE public.teacher (
id serial,
name varchar (255),
school_id int,
PRIMARY KEY (id),
FOREIGN KEY (school_id) REFERENCES  school(id)
);

INSERT INTO school (id, name) VALUES (generate_series(1, 150000), 'default');
INSERT INTO teacher (id, school_id, name) VALUES (generate_series(1, 350000), floor(random() * 150000) + 1, 'default');
INSERT INTO student (id, school_id, name) VALUES (generate_series(1, 15000000), floor(random() * 150000) + 1, 'default');
```

```sql
explain analyze select * from student where school_id = 3567;
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/06c50d65-f120-4001-b8a5-4183c4657c4e)

```sql
explain analyze select * from teacher where school_id = 3567;
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/6a294462-8141-4d53-ad14-50af79fbf5af)

```sql
EXPLAIN ANALYZE INSERT INTO student (id, school_id, name) VALUES (15000001, 3567, 'default');
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/fe80e523-ce44-4514-acdb-039efd2bc26f)

```sql
EXPLAIN ANALYZE INSERT INTO teacher (id, school_id, name) VALUES (350001, 3567, 'default');
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/ac0cf7a1-3f71-463a-adcc-b70559925f3a)

## Now, let's create indexes and see how it impacts query times.

```sql
CREATE INDEX student_school_id ON student (school_id);

CREATE INDEX teacher_school_id ON teacher (school_id);
```

```sql
explain analyze select * from student where school_id = 3567;
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/eb42acf8-09b2-4b85-81e6-794120b30dc1)

```sql
explain analyze select * from teacher where school_id = 3567;
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/effbf94b-1f02-476f-a3e6-c14bd97a5aca)

```sql
EXPLAIN ANALYZE INSERT INTO student (id, school_id, name) VALUES (15000001, 3567, 'default');
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/1a22e834-7d95-450a-9594-6844d76d5884)

```sql
EXPLAIN ANALYZE INSERT INTO teacher (id, school_id, name) VALUES (350001, 3567, 'default');
```
![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/fdb9e494-b7cc-49ba-8d59-82de7dcd68fb)

## Now, let's try to see response times in scenario where we have partitioned it on the basis of school_id

Note: You will have to use PostgreSQL with PgPartman Setup ([Dockerfile](https://gist.github.com/jgould22/3280fc0f531485f4fe19a2ef1ef67361))

```sql
CREATE TABLE public.school (
id serial,
PRIMARY KEY (id),
name varchar(255)
);

CREATE TABLE public.student (
id serial,
name varchar(255),
school_id int,
FOREIGN KEY (school_id) REFERENCES  school(id)
) PARTITION BY RANGE(school_id);

SELECT partman.create_parent('public.student', 'school_id', 'native', '1000');

CREATE TABLE public.teacher (
id serial,
name varchar (255),
school_id int,
FOREIGN KEY (school_id) REFERENCES  school(id)
) PARTITION BY RANGE(school_id);

SELECT partman.create_parent('public.teacher', 'school_id', 'native', '1000');

INSERT INTO school (id, name) VALUES (generate_series(1, 150000), 'default');

INSERT INTO teacher (id, school_id, name) VALUES (generate_series(1, 350000), floor(random() * 150000) + 1, 'default');

CALL partman.partition_data_proc('public.teacher');

INSERT INTO student (id, school_id, name) VALUES (generate_series(1, 15000000), floor(random() * 150000) + 1, 'default');

CALL partman.partition_data_proc('public.student');

CREATE INDEX student_school_id ON student (school_id);

CREATE INDEX teacher_school_id ON teacher (school_id);
```

```sql
explain analyze select * from student where school_id = 3567;
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/de051dc9-f813-4bda-b468-f5f34f6f9b96)

```sql
explain analyze select * from teacher where school_id = 3567;
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/9a895ae6-63f3-4b42-9142-db31ea4efa20)

```sql
EXPLAIN ANALYZE INSERT INTO student (id, school_id, name) VALUES (15000001, 3567, 'default');
```

![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/f69c545c-01aa-46ed-8b04-75d1e5b64803)

```sql
EXPLAIN ANALYZE INSERT INTO teacher (id, school_id, name) VALUES (350001, 3567, 'default');
```
![image](https://github.com/singhalkarun/postgresql-benchmarks/assets/113603846/63f8c754-c5ec-4601-9f8f-49bdc9327b2d)
