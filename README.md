# django-todo
A simple todo app built with django

![todo App](https://raw.githubusercontent.com/shreys7/django-todo/develop/staticfiles/todoApp.png)
### Setup
To get this repository, run the following command inside your git enabled terminal
```bash
$ git clone https://github.com/shreys7/django-todo.git
```

Second thing you can do is create a virtual Environment if you want to run the code in specific version of Python. 
For that you can follow some of the command to listed below:
 # virtualenv -p python3.7 env 
 # source env/bin/activate



You will need django to be installed in you computer to run this app. Head over to https://www.djangoproject.com/download/ for the download guide

# pip install django


Once you have downloaded django, go to the cloned repo directory and run the following command

# cd django-todo


```bash
$ python manage.py makemigrations
```

This will create all the migrations file (database migrations) required to run this App.

Now, to apply this migrations run the following command
```bash
$ python manage.py migrate
```

One last step and then our todo App will be live. We need to create an admin user to run this App. On the terminal, type the following command and provide username, password and email for the admin user
```bash
$ python manage.py createsuperuser
```

That was pretty simple, right? Now let's make the App live. We just need to start the server now and then we can start using our simple todo App. Start the server by following command

```bash
$ python manage.py runserver
```

Once the server is hosted, head over to http://127.0.0.1:8000/todos for the App.

Cheers and Happy Coding :)


A requirements.txt file in Django lists project dependencies, ensuring consistent installation across environments.

Command to create:
# pip freeze > requirement.txt

To install all the dependencies listed in the file in one go, use the following command:

# py


