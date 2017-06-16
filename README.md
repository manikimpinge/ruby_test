## Rapid River Software senior engineer position skills test

### 1. Voting
##### Each voter can vote in zero or more referenda. Each referendum has one or more questions, and each question is a yes/no vote. Write the simplest normalized schema to describe this in generic SQL statements or as an entity-relationship diagram. Point out where the indexes would be if you want to quickly know the results of a given referendum question, but you never expect to query a single voter's voting record.

Below is the rails Model code:

```sh
class Referendum < ActiveRecord::Base
  has_many :questions
end

class Question < ActiveRecord::Base
  belongs_to :referendum
  has_many :questions, through: "user_questions"
end

class User < ActiveRecord::Base
  has_many :questions, through: "user_questions"
end

class UserQuestion < ActiveRecord::Base
  belongs_to :question
  belongs_to :user
end
```
Check the attached screenshot for ER diagram.

https://www.dropbox.com/s/xhhw7i3vzg7oct9/20170616_153356.jpg?dl=0

### 2. Availability on AWS
##### Your client plans to host their web application on AWS. When would you advise (a) deployment in a single availability zone, (b) deployment in multiple availability zones in a single region ( c ) deployment in multiple regions? What operational difficulty or complexity, if any, is added as you go from (a) to (b) to ( c )?

I prefer multiple availability zones in a single region which is closer to specific customers you want to target. Availability zones in multiple locations protect applications from the failure of a single location so if one instance fails, there are other instances in other zones to handle the request.

- If move from (a) to (b), ie single to multiple availability zone, then we have to set the Auto Scaling of load balancer.

- If move from (b) to ( c ), ie single region to multiple region then all communication between regions is across the public Internet. Therefore, we should use the encryption methods to protect data and there is a charge for data transfer between regions also.

### 3. Lies, damn lies, and git
##### You're working on a cool branch of the foobar project, and your branch contains two commits A and B. 
The git lineage is:
 
```sh
X -- Y -- Z <-- master
           \
            A -- B <-- your-cool-branch
```
##### You want to contribute the code back to the master foobar repo, but you realize there is a really dumb typo in one of your source code comments in commit A. You'd still like to submit the pull request as two commits A' and B', where A' is the fixed version of A, and B' is the same exact diff as B. How do you rewrite git history to make this happen? Is B' the same hash as B? Why or why not?

I will follow below steps:
- First move to cool branch. Command: git checkout "cool branch"
- Then change the commit message of last commit ie "A". 
   Command: 
```sh
$ git commit --amend
```
   Above command open a text editor then will change the message and save the file and push to git then a new commit create on git.
- B' is same hash as B because we didn't change the message of B

### 4. Ruby metaprogramming
##### What's Module.included? What is a common (meta-)programming construct in which it is used? Give an example and explain exactly what's going on.

A Module is a collection of methods and constants. Module "included" method is called when you include module into a class, it is used for defining relations, scopes and validations. It gets called before you even have created object from that class.
for eg.
```sh
module A
  included do
    validates :attr, presence: true
    has_many :groups
  end
end

class B
  include A
end
```
- Modules "include" method are used to make mixins using which we can share behaviors between different classes and implement multiple inheritence in rails.

### 5. Someone did something dumb
##### New code was released to multi-tier production environment and now the site is SO slow. But it worked great on the single-server staging environment, which has only half the CPU and half the RAM of the production servers!
Here's the code that's behaving poorly:
```sh
class Post < ActiveRecord::Base
  def comments
    comment_ids = Comment.where("post_id = ?", self.id).map { |c| c.id }
    comment_ids.map { |comment_id| Comment.find_by_id(comment_id) }
  end
end
```
##### Q. What's wrong? And why did it work well (enough) on the staging server?
Following things are wrong in code:

query 1. comment_ids = Comment.where("post_id = ?", self.id).map { |c| c.id } 
#it calculates the ids of all comments and put it in array
query 2. comment_ids.map { |comment_id| Comment.find_by_id(comment_id) } 
#loop over the array and find Post comments one by one

It makes the process slow in multi-tier production environment because client and database are in different location. Client makes the request to server and server executes above code and fetch data from database. While in stagining envrinment, client and server are in single location thats why its executes it faster.

##### Q. Obviously the person who wrote this isn't a Rails programmer. How would a Rails programmer have written the code?
Yes, above code is not written by Rails programmer who can use association like:
```sh
class Post < ActiveRecord::Base
  has_many :comments
end

class Comment < ActiveRecord::Base
  belongs_to :post
end

#To fetch all comments for a Post:
class Post < ActiveRecord::Base
  def comments
    self.comments
  end
end
```
### 6. Unix tools
##### In one Unix command, find all of the files in /usr/local whose contents contain the word "aardvark" (case-insensitive), and list them all in order from most-recently created to least-recently created.

Command: 
```sh
find /usr/local -type f -iname "*aardvark*" -print
```
### 7. CSS
##### Usage of the "!important" CSS declaration is often frowned upon by front-end developers. What does the "!important" CSS declaration do? Why do front-end developers suggest using it with caution? What are some cases in which it makes sense to use it?

An !important declaration provides a way for a stylesheet to give a CSS value more weight than it has naturally.
eg.
```sh
#example {
	font-size: 14px !important;	
}
#container #example {
	font-size: 10px;
}
```
In the above code, the element with the id of "example" will have text sized at 14px, due to the addition of !important.

- Front-end developers suggest using it with caution because using !important, it overrides the css. It may work for some css but may be disrupts the css of others.

- There are some cases in which we have to use it like there will be CSS bugs on a live site and client wants to fix it quickly, then we can use !important to fix css issues quickly.

### 8. JavaScript
##### Discuss the advantages and disadvantages of using CoffeeScript in your JavaScript application.

Advantages:
- less characters to type
- the code is easier to read
- won't have to debug any missing "}" in your code
- strict use of indentation

Disadvantages:
- The lack of braces for function can be problem for some developer
- requires knowledge of Javascript before using it.
- can't run a debugger on it directly
