# Many-Students-in-many-Courses
This application will read roster data in JSON format, parse the file, and then produce an SQLite database that contains a User, Course, and Member table and populate the tables from the data file.

You can base your solution on this code: http://www.py4e.com/code3/roster/roster.py - this code is incomplete as you need to modify the program to store the role column in the Member table to complete the assignment.

Each student gets their own file for the assignment. Download this file and save it as roster_data.json. Move the downloaded file into the same folder as your roster.py program.

Once you have made the necessary changes to the program and it has been run successfully reading the above JSON data, run the following SQL command:

SELECT hex(User.name || Course.title || Member.role ) AS X FROM 
    User JOIN Member JOIN Course 
    ON User.id = Member.user_id AND Member.course_id = Course.id
    ORDER BY X
Find the first row in the resulting record set and enter the long string that looks like 53656C696E613333



SOLUTION:::
import sqlite3
import json
conn = sqlite3.connect('student.sqlite')
curr = conn.cursor()
sql_command = """DROP TABLE IF EXISTS User;
DROP TABLE IF EXISTS Course;
DROP TABLE IF EXISTS Member;

	Create table User(
		id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
		name TEXT
	);

	create table Course(
		id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
		title TEXT
	);
	create table Member(
		user_id INTEGER,
		course_id INTEGER,
		role INTEGER,
		primary key(user_id, course_id)
	);

	
	"""
curr.executescript(sql_command)
handle = open("roster_data.json")
data = handle.read()
str_data = json.loads(data)
for entry in str_data:
	name = entry[0]
	title = entry[1]
	role = entry[2]
	#print((name, entry))
	curr.execute("""
		Insert or ignore into User (name)
		values(?)""", (name,))
	curr.execute("""
		select id from User where name = ?""", (name,))
	user_id = curr.fetchone()[0]
	curr.execute("""
		insert or ignore into Course(title)
		values(?)""",(title,))
	curr.execute("""select id from Course where title = ?""",(title,))
	course_id = curr.fetchone()[0]
	curr.execute("""insert or replace into Member(user_id, course_id,role) 
		values(?,?,?)""",(user_id,course_id,role))
	
	conn.commit()
curr.execute("""SELECT hex(User.name || Course.title || Member.role ) AS X FROM 
User JOIN Member JOIN Course 
ON User.id = Member.user_id AND Member.course_id = Course.id
ORDER BY X""")
print(curr.fetchone())
