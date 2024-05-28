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

## Development

### Login/Register with hashed password
The user
```.py
@app.route('/', methods=['GET','POST'])
def login():  # put application's code here
    error_text=""
    if request.method == 'POST':
        uname = request.form.get('uname')
        password = request.form.get('psw')
        db = DatabaseBridge('database.db')
        results = db.search(query='Select * from users', multiple=True)
        db.close()
        print(results)
        for row in results:
            if uname == row[1]:
                if check_hash (row[2], password):
                    user_id = row[0]
                    session['user_id'] = user_id
                    print("user correctly logged in")
                    return redirect(url_for('index'))
                else:
                    error_text = "Password does not match. Please try again."
            else:
                error_text = "No such username. Please try again."

    return render_template('login.html', error=error_text)
@app.route('/register', methods=['GET','POST'])
def register():
    if not session.get('user_id'):
        return redirect(url_for('login'))
    user_id = session['user_id']
    db = DatabaseBridge('database.db')
    error_text = ""
    if request.method == 'POST':
        uname = request.form.get('uname')
        password = request.form.get('psw')
        confirm_pass = request.form.get('psw_confirm')

        # Check if username already exists
        existing_user = db.search(query=f"SELECT * FROM users WHERE user_name = '{uname}'", multiple=False)
        if existing_user:
            error_text = "Username already taken. Please choose another username."
        elif password != confirm_pass:
            error_text = "Passwords do not match. Please try again."
        else:
            hashed_pass = make_hash(password)
            db.run_query(query=f"INSERT INTO users (user_name, user_pass) VALUES ('{uname}', '{hashed_pass}')")
            return redirect(url_for('register_success'))

    return render_template('register.html', error=error_text)

@app.route('/register/success', methods=['GET','POST'])
def register_success():

    return render_template('Register_success.html')
```

### Home screen
```.py
@app.route('/home')
def index():
    if not session.get('user_id'):
        return redirect(url_for('login'))

    user_id = session['user_id']
    db = DatabaseBridge('database.db')
    foodfollow = db.search(query=f"SELECT followfood FROM users WHERE id={user_id}",
                      multiple=False)[0]
    if foodfollow==0:
        food_follow = "FOLLOW?"
        my_class_food = 'btn-info'
    else:
        food_follow = "FOLLOWED"
        my_class_food = 'btn-primary'
    moviefollow = db.search(query=f"SELECT followmovie FROM users WHERE id={user_id}",
                           multiple=False)[0]
    if moviefollow == 0:
        movie_follow = "FOLLOW?"
        my_class = 'btn-info'
    else:
        movie_follow = "FOLLOWED"
        my_class = 'btn-primary'


    query = f"""SELECT user_name FROM users"""
    users_list = db.search(query=query, multiple = True)
    user_list = []
    print('users_list',users_list)

    for u in users_list:
        print(u)

        user_list += u
    print('ok',user_list)
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

    return render_template('index.html', users=user_list_html, my_class=my_class_food,myclass=my_class,food_follow = food_follow, movie_follow=movie_follow, followed_user=followed_users)

```
### Accessing Categories
```.py
@app.route('/food/all')
def get_all_food():
    if not session.get('user_id'):
        return redirect(url_for('login'))
    user_id  = session['user_id']
    db = DatabaseBridge('database.db')
    result = db.search(query=f'Select user_name from users where id ={user_id}', multiple=True)
    user = result

    db = DatabaseBridge('database.db')
    results = db.search(query = 'Select * from foods', multiple=True)
    db.close()
    return render_template('foods.html',foods=results, name=user)

@app.route('/food/<int:food_id>', methods={'GET','POST'})
def get_food(food_id):
    if not session.get('user_id'):
        return redirect(url_for('login'))

    user_id = session['user_id']
    db = DatabaseBridge('database.db')
    liked = db.search(query=f"SELECT * FROM likes WHERE food_id={food_id} AND user_id={user_id}",
                      multiple=False)
    print(liked)
    if liked:
        like = "liked"
        my_class = 'btn-info'
    else:
        like = "like?"
        my_class = 'btn-primary'

    f = db.search(query=f"Select * from foods where id={food_id}", multiple=False)
    print(f)
    reviews = db.search(query=f"Select * from reviews where food_id={food_id}", multiple=True)
    print(reviews)
    if request.method == 'POST':
        if 'submit' in request.form:
            #the form was submitted with the new review
            date = datetime.now().strftime('%Y/%b/%d')
            comment = request.form.get('comment')
            stars = request.form.get('stars')
            query = f"""INSERT into reviews(date,comment,stars,food_id) values('{date}','{comment}',{stars},{food_id})"""
            print(query)
            db.insert(query=query)
            db.close()
            return redirect(url_for('get_food',food_id=food_id, like=like))


    return render_template('food.html', food=f, reviews = reviews, like=like, my_class=my_class)
```

### Edit/Create/Delete comment
```.py
@app.route('/food/<int:food_id>/review/<int:review_id>/edit', methods=['GET','POST'])
def edit_review(review_id,food_id):
    if not session.get('user_id'):
        return redirect(url_for('login'))
    user_id = session['user_id']
    db = DatabaseBridge('database.db')
    f = db.search(query=f"Select * from foods where id={food_id}", multiple=False)
    # print(f)
    reviews = db.search(query=f"Select * from reviews where food_id={food_id}", multiple=True)
    review_to_edit = db.search(query=f"Select * from reviews where id={review_id}", multiple=False)
    comments = review_to_edit[1]
    stars = review_to_edit[3]
    if request.method == 'POST': #update the review and redirect to the food page
        comment = request.form.get('comment')
        stars = request.form.get('stars')
        db.run_query(f"UPDATE reviews set comment='{comment}', stars={stars} where id = {review_id}")
        db.close()
        return redirect(url_for('get_food',food_id=food_id))

    db.close()
    return render_template('food.html',food=f,reviews=reviews, comments=comments, stars=stars)

@app.route('/food/<int:food_id>/review/<int:review_id>/delete')
def delete_review(review_id, food_id):
    if not session.get('user_id'):
        return redirect(url_for('login'))
    user_id = session['user_id']
    db = DatabaseBridge('database.db')
    db.run_query(f"DELETE from reviews where id = {review_id}")
    db.close()
    return redirect(url_for('get_food',food_id=food_id))
```

### Add/remove like
```.py
@app.route('/like/<int:food_id>',methods=['GET','POST'])
def like(food_id):
    if not session.get('user_id'):
        return redirect(url_for('login'))
    user_id = session['user_id']
    db = DatabaseBridge('database.db')
    liked = db.search(query=f"SELECT * FROM likes WHERE food_id={food_id} AND user_id={user_id}",
                      multiple=False)
    print(liked)
    if liked:
        # If already liked, remove the like
        db.run_query(query=f"DELETE FROM likes WHERE food_id={food_id} AND user_id={user_id}")
        like = "like?"
    else:
        # If not liked, add the like
        db.insert(query=f"INSERT INTO likes (food_id, user_id) VALUES ({food_id}, {user_id})")
        like = "liked"
    return redirect(url_for('get_food', food_id=food_id, like=like))
```

### Follow user/categories
```.py
@app.route('/follow/food',methods=['GET','POST'])
def follow():
    if not session.get('user_id'):
        return redirect(url_for('login'))
    user_id = session['user_id']
    db = DatabaseBridge('database.db')
    liked = db.search(query=f"SELECT followfood FROM users WHERE id={user_id}",
                      multiple=False)[0]
    print(liked,'here')
    if liked == 1:
        # If already liked, remove the like
        db.run_query(query=f"Update users set followfood = 0  where id={user_id}")
        followfood = "FOllOW?"
    else:
        # If not liked, add the like
        db.run_query(query=f"Update users set followfood = 1  where id={user_id}")
        followfood = "FOLLOWED"

    return redirect(url_for('index',food_follow = followfood))

@app.route('/follow/user/', methods=['GET', 'POST'])
def follow_user():
    followed_user = request.args.get('followed_user')
    print('followed_user',followed_user)
    db = DatabaseBridge('database.db')
    followed_id = db.search(query=f"select id from users where user_name='{followed_user}'")[0]
    if not session.get('user_id'):
        return redirect(url_for('login'))
    user_id = session['user_id']
    liked = db.search(query=f"SELECT * FROM following WHERE followed = {followed_id} and following={user_id}",
                      multiple=False)
    if liked == None:
        db.run_query(query=f"INSERT INTO following (following, followed) VALUES ({user_id}, {followed_id})")
    else:
        db.run_query(query=f'delete from following where following = {user_id} and followed = {followed_id}')
    return redirect(url_for('index'))
```
### Profile Page
```.py
@app.route('/profile')
def profile():
    if not session.get('user_id'):
        return redirect(url_for('login'))
    user_id = session['user_id']
    db = DatabaseBridge('database.db')
    user_me = db.search(query=f'select user_name from users where id={user_id}')
    following = db.search(query=f'Select followed from following where following = {user_id}', multiple=True)
    followed_users = []
    # print(following)
    for user in following:

        # print(user[0])
        followed_users += db.search(query=f'select user_name from users where id={int(user[0])}')
    print(followed_users)

    following_categories= ''
    movie = db.search(query=f'select followmovie from users where id = {user_id}', multiple= False)[0]
    if movie == 1:
        following_categories += 'movie'
    food = db.search(query=f'select followfood from users where id={user_id}',multiple=False)[0]
    if food ==1:
        if following_categories != '':
            following_categories += ' and '
        following_categories += 'food'
    return render_template('profile.html',name=user_me, following_users = followed_users, following_categories=following_categories)

```

# Criteria D
A 5 min video on the functionality of the code

https://drive.google.com/file/d/1QkHsz-RAvzNqk_of141M763ehVTX9XVG/view?usp=sharing

# Criteria E
# Evaluation of the product
In order to improve the usability, I made sure the design was consistent with intentional color coordinations and font consistency.
Moreover, the tables used in the home page where users can navigate to view all the movies/foods or when viewing all the movies/foods allows users to understand the page clearly and allows organization of the page.

Furthermore, the simplicity of the design allows users of any age or technology familiarity to navigate through the system easily.

# Recommendations for further improvement
Below are some other functions that could be added for improvement
* Posting images
* Sending Emails
* Forgot Password/Change Password functions
* Adding categories
* Viewing liked posts/topics
