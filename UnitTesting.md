### Step 1 — Adding a Test Suite to the Django Application
A test suite in Django is a collection of all the test cases in all the apps in your project. To make it possible for the Django testing utility to discover the test cases you have, you write the test cases in scripts whose names begin with test. In this step, you’ll create the directory structure and files for your test suite, and create an empty test case in it.

If you followed the Django Development tutorial series, you’ll have a Django app called blogsite.

Let’s create a folder to hold all our testing scripts. First, activate the virtual environment:
```
cd ~/my_blog_app
. env/bin/activate
``` 
Then navigate to the blogsite app directory, the folder that contains the `models.py` and `views.py` files, and then create a new folder called tests:
```
cd ~/my_blog_app/blog/blogsite
mkdir tests
``` 
Next, you’ll turn this folder into a Python package, so add an __init__.py file:
```
cd ~/my_blog_app/blog/blogsite/tests
touch __init__.py
``` 
You’ll now add a file for testing your models and another for testing your views:
```
touch test_models.py
touch test_views.py
``` 
Finally, you will create an empty test case in test_models.py. You will need to import the Django TestCase class and make it a super class of your own test case class. Later on, you will add methods to this test case to test the logic in your models. Open the file test_models.py:
```
nano test_models.py
```
Now add the following code to the file:
```
~/my_blog_app/blog/blogsite/tests/test_models.py
from django.test import TestCase

class ModelsTestCase(TestCase):
    pass
```
You’ve now successfully added a test suite to the blogsite app. Next, you will fill out the details of the empty model test case you created here.

### Step 2 — Testing Your Python Code
In this step, you will test the logic of the code written in the models.py file. In particular, you will be testing the save method of the Post model to ensure it creates the correct slug of a post’s title when called.

Let’s begin by looking at the code you already have in your models.py file for the save method of the Post model:

cd ~/my_blog_app/blog/blogsite
nano models.py
 
You’ll see the following:
```
~/my_blog_app/blog/blogsite/models.py
class Post(models.Model):
    ...
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        super(Post, self).save(*args, **kwargs)
    ...
``` 
We can see that it checks whether the post about to be saved has a slug value, and if not, calls slugify to create a slug value for it. This is the type of logic you might want to test to ensure that slugs are actually created when saving a post.

Close the file.

To test this, go back to test_models.py:
```
nano test_models.py
``` 
Then update it to the following, adding in the highlighted portions:
```
~/my_blog_app/blog/blogsite/tests/test_models.py
from django.test import TestCase
from django.template.defaultfilters import slugify
from blogsite.models import Post


class ModelsTestCase(TestCase):
    def test_post_has_slug(self):
        """Posts are given slugs correctly when saving"""
        post = Post.objects.create(title="My first post")

        post.author = "John Doe"
        post.save()
        self.assertEqual(post.slug, slugify(post.title))
``` 
This new method test_post_has_slug creates a new post with the title "My first post" and then gives the post an author and saves the post. After this, using the assertEqual method from the Python unittest module, it checks whether the slug for the post is correct. The assertEqual method checks whether the two arguments passed to it are equal as determined by the "==" operator and raises an error if they are not.

Save and exit `test_models.py`.

This is an example of what can be tested. The more logic you add to your project, the more there is to test. If you add more logic to the save method or create new methods for the Post model, you would want to add more tests here. You can add them to the test_post_has_slug method or create new test methods, but their names must begin with test.

You have successfully created a test case for the Post model where you asserted that slugs are correctly created after saving. In the next step, you will write a test case to test views.

### Step 3 — Using Django’s Test Client
In this step, you will write a test case that tests a view using the Django test client. The test client is a Python class that acts as a dummy web browser, allowing you to test your views and interact with your Django application the same way a user would. You can access the test client by referring to self.client in your test methods. For example, let us create a test case in test_views.py. First, open the test_views.py file:
```
nano test_views.py
```
Then add the following:
```
~/my_blog_app/blog/blogsite/tests/test_views.py
from django.test import TestCase


class ViewsTestCase(TestCase):
    def test_index_loads_properly(self):
        """The index page loads properly"""
        response = self.client.get('your_server_ip:8000')
        self.assertEqual(response.status_code, 200)
``` 
The `ViewsTestCase` contains a `test_index_loads_properly` method that uses the Django test client to visit the index page of the website (http://your_server_ip:8000, where your_server_ip is the IP address of the server you are using). Then the test method checks whether the response has a status code of 200, which means the page responded without any errors. As a result you can be sure that when the user visits, it will respond without errors too.

Apart from the status code, you can read about other properties of the test client response you can test in the Django Documentation Testing Responses page.

In this step, you created a test case for testing that the view rendering the index page works without errors. There are now two test cases in your test suite. In the next step you will run them to see their results.

### Step 4 — Running Your Tests
Now that you have finished building a suite of tests for the project, it is time to execute these tests and see their results. To run the tests, navigate to the blog folder (containing the application’s manage.py file):
```
cd ~/my_blog_app/blog
``` 
Then run them with:
```
python manage.py test
``` 
You’ll see output similar to the following in your terminal:
```
Output
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.007s

OK
Destroying test database for alias 'default'...
```
In this output, there are two dots .., each of which represents a passed test case. Now you’ll modify test_views.py to trigger a failing test. First open the file with:
```
nano test_views.py
``` 
Then change the highlighted code to:
```
~/my_blog_app/blog/blogsite/tests/test_views.py
from django.test import TestCase


class ViewsTestCase(TestCase):
    def test_index_loads_properly(self):
        """The index page loads properly"""
        response = self.client.get('your_server_ip:8000')
        self.assertEqual(response.status_code, 404)
``` 
Here you have changed the status code from 200 to 404. Now run the test again from your directory with manage.py:
```
python manage.py test
``` 
You’ll see the following output:
```
Output
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.F
======================================================================
FAIL: test_index_loads_properly (blogsite.tests.test_views.ViewsTestCase)
The index page loads properly
----------------------------------------------------------------------
Traceback (most recent call last):
  File "~/my_blog_app/blog/blogsite/tests/test_views.py", line 8, in test_index_loads_properly
    self.assertEqual(response.status_code, 404)
AssertionError: 200 != 404

----------------------------------------------------------------------
Ran 2 tests in 0.007s

FAILED (failures=1)
Destroying test database for alias 'default'...
```
You see that there is a descriptive failure message that tells you the script, test case, and method that failed. It also tells you the cause of the failure, the status code not being equal to 404 in this case, with the message AssertionError: 200 != 404. The AssertionError here is raised at the highlighted line of code in the test_views.py file:
```
~/my_blog_app/blog/blogsite/tests/test_views.py
from django.test import TestCase


class ViewsTestCase(TestCase):
    def test_index_loads_properly(self):
        """The index page loads properly"""
        response = self.client.get('your_server_ip:8000')
        self.assertEqual(response.status_code, 404)
``` 
It tells you that the assertion is false, that is, the response status code (200) is not what was expected (404). Preceding the failure message, you can see that the two dots .. have now changed to .F, which tells you that the first test case passed while the second didn’t.
