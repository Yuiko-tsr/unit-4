# Introduction
This quarter, we were tasked to create a Reddit kind of website where users can post, edit, delete comments and follow topics of their choice. I created two categories, food and movie, with 50 contents in each.

## Success Criteria
* A login/registration system, hashed of course.
* A posting system to EDIT/CREATE/DELETE comments.
* A system to add/remove likes.
* A system to follow/unfollow users, follow/unfollow topics or groups.
* A profile page with relevant information

# Criteria C
## Tools used
* HTTP server
* POST, GET Request
* SQLite databases 
* Functions
* For loops
* If statements
* Variables
* Encryption
* Nesting loops
* Cookie
* CSS designs

## Sources
1. For login systems I used a template: https://colorlib.com/wp/html5-and-css3-login-forms/ 
2. For button designes I used template: https://www.cirrus-ui.com/buttons/button-groups
3. Pochaco icon for the login page: https://www.sanrio.com/collections/pochacco
4. Chat GPT for all the content of food and movies
5. For profile page I used a template: https://themeforest.net/category/site-templates?term=profile%20page
6. https://tadabase.io/blog/using-join-tables

## Development

### Register Page
Here we have two key features. First is to comapre the entered username with the already registered username as two users with the same username could cause an issue with the profile or the follow user system. THis can easily be solved with the code below that first takes out all the usernames in the database and gives an error text if it doesn't apply.

```.py
existing_user = db.search(query=f"SELECT * FROM users WHERE user_name = '{uname}'", multiple=False)
if existing_user:
    error_text = "Username already taken. Please choose another username."
```

The second key feature is hashing the password when saving it into the database. Once the username has been confirmed, the user will be added into the databse. When doing so, we must make sure the password is hashed to maintain the security of the user information from malcious software. This can be done using the **make_hash** function below:
```.py
def make_hash(text:str):
    return hasher.hash(text)
```
This is applied in the code below where once the username is confirmed and the passwords is also confirmed, we proceed into creating the hashed text and entering it into the database.
```.py
existing_user = db.search(query=f"SELECT * FROM users WHERE user_name = '{uname}'", multiple=False)
if existing_user:
    error_text = "Username already taken. Please choose another username."
elif password != confirm_pass:
    error_text = "Passwords do not match. Please try again."
else:
    hashed_pass = make_hash(password)
    db.run_query(query=f"INSERT INTO users (user_name, user_pass) VALUES ('{uname}', '{hashed_pass}')")
    return redirect(url_for('register_success'))

```
### Login Page
Firstly in the login page we placed the password as **type="password"** to encrypt it when it is typed. This helps protect the user information when it is being entered by the user and prevents other people from physically seeing the typed password. This can be seen in the code below:
HTML
```.py
<label for="psw"><b>Password</b></label>
<input type="password" placeholder="Enter Password" name="psw" required>
```

Moreover, since the password has beeen hashed in the database, when loging in they must make sure that the hashed password is the same as the entered password. We can do this by using the pycharm code below:
```.py
def check_hash(input_hash, text):
    return hasher.verify(text, input_hash)
```

In this code we can input hashed text and normal text and compare them to make sure they are the same. By using this we can successfully compare the two types of texts without an issue. This is applied in the code below:
```.py
if uname == row[1]:
    if check_hash (row[2], password):
        user_id = row[0]
        session['user_id'] = user_id
        print("user correctly logged in")
        return redirect(url_for('index'))
    else:
        error_text = "Password does not match. Please try again."
```

### Accessing Categories
Here is the HTML for the food category screeen where you can see all the food in a table format with their ingredients, cusine, price and name. This table format allows users to easily view the list of foods with clarity and ease. Here I used a loop of the database and printed each  of the information.

```.py
<table class="table">
    <tr>
        <th>No</th>
        <th>Name</th>
        <th>Ingredients</th>
        <th>Cusine</th>
        <th>Price ($)</th>
        <th>View More</th>
    </tr>
    {% for f in foods %}
    <tr>
        <td>{{ loop.index }}</td>
        <td>{{ f[1] }}</td>
        <td>{{ f[2] }}</td>
        <td>{{ f[4] }}</td>
        <td>{{ f[3] }}</td>
        <td><a class="btn btn-primary" href="{{ url_for('get_food',food_id=f[0]) }}">View More</a></td>
    </tr>
    {% endfor %}
</table>
```


### Follow user/categories
For the follow user system, I created a joined table that allows me to connect one user table to another without difficulty. Joined tables in a database are a way to combine data from multiple related tables into a single table. They are used to retrieve data that is spread across different tables, establish relationships between tables, normalize data by dividing it into multiple tables to avoid redundancy, and combine data from different sources or databases. [6] I used the **foreign key** variable to make sure that the variables were clearly associated to the user id and by joining tables, I was able to see which user followed which other user. The SQLite code is below:
```.py
create table if not exists following(
    id integer primary key,
    following integer,
    followed integer,
    foreign key (following) references users(id),
    foreign key (followed) references users(id)
);
```
After creating a table with **foreign key** I moved onto the pycharm code where I would look through this table and allow for the buttons on the HTML to follow or unfollow users according to their status. Therefore, I used an if statement where I asked whether the user had followed that specific user on the website and if they had the button would delete the followed history where it deletes the row of the two users from the database and if they hadnt followed they would add the followed history onto the database.

```.py
@app.route('/follow/user/', methods=['GET', 'POST'])
def follow_user():
    followed_user = request.args.get('followed_user')
    followed_id = db.search(query=f"select id from users where user_name='{followed_user}'")[0]
    user_id = session['user_id']
    liked = db.search(query=f"SELECT * FROM following WHERE followed = {followed_id} and following={user_id}",
                      multiple=False)
    if liked == None:
        db.run_query(query=f"INSERT INTO following (following, followed) VALUES ({user_id}, {followed_id})")
    else:
        db.run_query(query=f'delete from following where following = {user_id} and followed = {followed_id}')
    return redirect(url_for('index'))
```

Moreover, to make things more visibly clear, I also added color specifications of whether the user had been followed or not. This can be seen below in the Pycharm code and later in the HTML code where I use another if statement where if the user had been followed the button class becomes red and the screen prints the words **FOLLOWED** while if they are not followed the button class becomes blue and the screen prints the words **FOLLOW?**. This design allows the users to visibly see their status with other users.

Pycharm:
```.py
 following = db.search(query=f'Select followed from following where following = {user_id}', multiple=True)
followed_users = ''
# print(following)
for user in following:
    if followed_users !='':
        followed_users += ','
    # print(user[0])
    followed_users += db.search(query=f'select user_name from users where id={int(user[0])}')[0]


print(followed_users)
user_list_html=[]
for u in user_list:
    user_list_html.append([u, u in followed_users])
```

HTML:
```.py
{% for user,status in users %}
    <li>{{ user }}</li>
    {% if status == True %}
    <a class="btn btn-primary" href="{{url_for('follow_user', followed_user=user)}}">Followed</a>
    {% else %}
    <a class="btn btn-info" href="{{url_for('follow_user', followed_user=user)}}">Follow?</a>
    {% endif %}
{% endfor %}
```
Above is the code that devides the users into followed or not followed and they will change the color fo the button as seen **class="btn btn-primary"** and **class="btn btn-info"** according to the status of the user that is specified in the pycharm by the if statement.

### Profile Page
For this profile page, initially the html showed all users in one line. However, this was unpracticle as the number of users following increased, the line got longer and was hard to read. Therefore, I put the users in a for loop and for each user I printed them out as a list. This can be seen from the code bellow:
```.py
<ul class="follow-list">
    {% for user in following_users %}
        <li>{{ user }}</li>

    {% endfor %}
</ul>
```
by using the curly brackets in the HTML I was able to improve the usability of the page by making it more clear and easy to read.

However, as for the categories, since we only have two I put them in one row as the design was illogical when the user had only followed one category. This is because bulletpoints usually consist of more than one row but when a user only follows one category it is only one row. Therefore, I put them in one row, separating them with an 'and' in the pycharm code and printing it directly in the HTML as shown bellow.
HTML:
```.py
<p><strong>Followed Categories:</strong></p>
<p style="text-indent: 60px;">{{ following_categories }}</p>
```

Pycharm:
```.py
following_categories= ''
movie = db.search(query=f'select followmovie from users where id = {user_id}', multiple= False)[0]
if movie == 1:
    following_categories += 'movie'
food = db.search(query=f'select followfood from users where id={user_id}',multiple=False)[0]
if food ==1:
    if following_categories != '':
        following_categories += ' and '
    following_categories += 'food'
```
In this Pycharm code I first created a variable called **following_categories** and added any categories that had been followed by the user. This way, I can easily print them out by sending the **following_categories** to the HTML which I do here: **'<p style="text-indent: 60px;">{{ following_categories }}</p>'**

# Criteria D
A 5 min video on the functionality of the code

https://drive.google.com/file/d/1QkHsz-RAvzNqk_of141M763ehVTX9XVG/view?usp=sharing

# Criteria E
# Evaluation of the product
In order to improve the usability, I made sure the design was consistent with intentional color coordinations and font consistency.
Moreover, the tables used in the home page where users can navigate to view all the movies/foods or when viewing all the movies/foods allows users to understand the page clearly and allows organization of the page.

Furthermore, the simplicity of the design allows users of any age or technology familiarity to navigate through the system easily.

# Beta testing
**To user**
* This is an application that allows you to view comment on food and movies and add comments or delete comments. You can also follow or unfollow users and categories. Please test for easiness of use and any bugs that may occur.

**User Feedback**
* Marina Mendieta testing (Beta testing by developer):
    * Good UI, very promising if expanded further, aditionally it shows versatility since it has both foods and movies. I would add images for every post and possibly a posting system for the user.
    * You should make sure that the comments can only be edited and deleted by the user that posted it. 
* Miki Hayashi testing (Beta testing by non-developer):
    * Easy to navigate, clear design, maybe for the further improvement you could add on the all food page to see whether I have liked the specific category or not.

# Recommendations for further improvement
Below are some other functions that could be added for improvement in the future
* Posting images
* Sending Emails
* Forgot Password/Change Password functions
* Adding categories
* Viewing liked posts/topics
