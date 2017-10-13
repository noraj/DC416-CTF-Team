# Needle in a haystack

## Description
We infiltrated Evil Robot Corp’s network and were able to get this partial data dump of one of their production hosts before their Android Monitoring Systems kicked us out. Can you do anything with it? We know they aren't good at developer best practices when doing app development.

### Solution

Let's analyze what this file is:
```
~# file needle-in-the-haystack
needle-in-the-haystack: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "mkfs.fat", sectors/cluster 4, reserved sectors 4, root entries 512, sectors 10240 (volumes <=32 MB) , Media descriptor 0xf8, sectors/FAT 8, sectors/track 63, heads 255, serial number 0x1e92c764, unlabeled, FAT (12 bi)
```

So this is is recognized as a DOS/MBR boot file. let's see what we can extract from it with basic strings.

```
~# strings needle-in-a-haystack

... grabage...

0000000000000000000000000000000000000000 9331ed8a2c47b3fd753d60d57b796ebacfb4d69a Sarah Harvey <shh@squareup.com> 1502560071 -0700	commit (initial): Initial commit
9331ed8a2c47b3fd753d60d57b796ebacfb4d69a 6a5f0a68319d5aa753f0655aeba70c7393fe7438 Sarah Harvey <shh@squareup.com> 1502560367 -0700	commit: Add welcome controller
6a5f0a68319d5aa753f0655aeba70c7393fe7438 5b4bafa1a476a7fa1cdc83a87e3a7b1019c75841 Sarah Harvey <shh@squareup.com> 1502560407 -0700	commit: Change welcome index text
5b4bafa1a476a7fa1cdc83a87e3a7b1019c75

... garbage ...

blog/.gitignore
blog/Gemfile
blog/Gemfile.lock
blog/README.rdoc
blog/Rakefile
blog/app/assets/images/.keep


... garbage ...
ref: refs/heads/master
Add security
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch security
# Changes to be committed:
#	modified:   app/controllers/articles_controller.rb
#	modified:   app/controllers/comments_controller.rb
# Untracked files:
#	config/database.yml
#	config/secrets.yml
```

So it looks like we have some git commits, some Ruby web application code and interesting untracked files like database.yml and secrets.yml, maybe they contain something about the flag?

```
~# fls /tmp/needle-in-the-haystack
r/r * 4:    deploy.tar.gz
d/d 6:    blog
d/d * 8:    .git
v/v 163523:    $MBR
v/v 163524:    $FAT1
v/v 163525:    $FAT2
d/d 163526:    $OrphanFiles
```

so we have a tarball called deploy at inode 4, the blog at inode 6 and a git repo at inode 8, let's extract the tarball with icat and decompress it.

```
~# icat needle-in-the-haystack 4 > deploy.tar.gz
~# tar xzvf deploy.tar.gz -C .

-r--r-- s/users         486 2017-08-12 13:46 blog/config/initializers/assets.rb
-rw-r--r-- s/users         647 2017-08-12 13:46 blog/config/initializers/inflections.rb
-rw-r--r-- s/users         136 2017-08-12 13:46 blog/config/initializers/session_store.rb
-rw-r--r-- s/users         194 2017-08-12 13:46 blog/config/initializers/filter_parameter_logging.rb
-rw-r--r-- s/users         500 2017-08-12 13:46 blog/config/initializers/to_time_preserves_timezone.rb
-rw-r--r-- s/users        1111 2017-08-12 13:46 blog/config/application.rb
-rw-r--r-- s/users         132 2017-08-12 13:46 blog/config/boot.rb
drwxr-xr-x s/users           0 2017-08-12 15:22 .git/
-rw-r--r-- s/users        9977 2017-08-12 14:51 .git/index
-rw-r--r-- s/users          23 2017-08-12 14:51 .git/HEAD
-rw-r--r-- s/users         368 2017-08-12 14:50 .git/COMMIT_EDITMSG
drwxr-xr-x s/users           0 2017-08-12 15:22 .git/logs/
-rw-r--r-- s/users        9631 2017-08-12 14:51 .git/logs/HEAD
drwxr-xr-x s/users           0 2017-08-12 13:47 .git/logs/refs/
drwxr-xr-x s/users           0 2017-08-12 14:49 .git/logs/refs/heads/
-rw-r--r-- s/users         316 2017-08-12 14:47 .git/logs/refs/heads/refactoring
-rw-r--r-- s/users         650 2017-08-12 14:44 .git/logs/refs/heads/comments
-rw-r--r-- s/users        4169 2017-08-12 14:51 .git/logs/refs/heads/master
-rw-r--r-- s/users         311 2017-08-12 14:50 .git/logs/refs/heads/security
-rw-r--r-- s/users         480 2017-08-12 14:49 .git/logs/refs/heads/deleting

```

Interesting, so there are a few branches named master, security, deleting, refactoring and also application files.

I recursively grepped for the word flag-\* in the web application tree but found nothing.

Let's examine the git repo.

```
root@toxicity:/tmp/tt# git branch
  comments
  deleting
* master
  refactoring
  security
root@toxicity:/tmp/tt# git log
commit 70659e69609dd836810eaa46bdd37725c01afc6d
Author: Sarah Harvey <shh@squareup.com>
Date:   Sat Aug 12 11:50:56 2017 -0700

    Add security

commit 3302b4e931c3b83d407785c110e752d42d8d6943
Author: Sarah Harvey <shh@squareup.com>
Date:   Sat Aug 12 11:44:03 2017 -0700

    Add comment showing


commit fe4d8eb7bce7b7594f58a6becf89a4c18476a8ef
Author: Sarah Harvey <shh@squareup.com>
Date:   Sat Aug 12 11:29:54 2017 -0700

    Add some article validation

commit d8478c26aa742bc1cd0311d9c720a83336ed2cc9
Author: Sarah Harvey <shh@squareup.com>
Date:   Sat Aug 12 11:27:59 2017 -0700

    Add links

... more commits ...

```

the 'Add Security' commit is interesting, let's find out what was pushed 

```
~# git show 70659e69609dd836810eaa46bdd37725c01afc6d
commit 70659e69609dd836810eaa46bdd37725c01afc6d
Author: Sarah Harvey <shh@squareup.com>
Date:   Sat Aug 12 11:50:56 2017 -0700

    Add security

diff --git a/blog/app/controllers/articles_controller.rb b/blog/app/controllers/articles_controller.rb
index 61e5d45..951a247 100644
--- a/blog/app/controllers/articles_controller.rb
+++ b/blog/app/controllers/articles_controller.rb
@@ -1,4 +1,6 @@
 class ArticlesController < ApplicationController
+  http_basic_authenticate_with name: "shh", password: "flap-12b36a752f93870393d8311b4e1529c1", except: [:index, :show]
+
   def index
     @articles = Article.all
   end
diff --git a/blog/app/controllers/comments_controller.rb b/blog/app/controllers/comments_controller.rb
index 80c120a..e0773f0 100644
--- a/blog/app/controllers/comments_controller.rb
+++ b/blog/app/controllers/comments_controller.rb
@@ -1,4 +1,6 @@
 class CommentsController < ApplicationController
+  http_basic_authenticate_with name: "shh", password: "flap-12b36a752f93870393d8311b4e1529c1", only: :destroy
+
   def create
     @article = Article.find(params[:article_id])
     @comment = @article.comments.create(comment_params)

```

We have a troll flag named flap, that's not what we want.

Eventually after exploring the refs/heads of the branches, we found secrets.yaml was changed.

```
~# /tmp/.git/logs/refs/heads# cat master
.. Sarah Harvey <shh@squareup.com> 1502560071 -0700    commit (initial): Initial commit
.. Sarah Harvey <shh@squareup.com> 1502560367 -0700    commit: Add welcome controller
.. Sarah Harvey <shh@squareup.com> 1502560407 -0700    commit: Change welcome index text
.. Sarah Harvey <shh@squareup.com> 1502560494 -0700    commit: Add route
.. Sarah Harvey <shh@squareup.com> 1502560953 -0700    commit: Basic article add
68490b1f703f667c2dbf9032104a2dcd1039204e 50cfa56ae51be4ca053a5fcea5589a621d982c6c Sarah Harvey <shh@squareup.com> 1502561092 -0700    rebase -i (finish): refs/heads/master onto
.. Sarah Harvey <shh@squareup.com> 1502563747 -0700    merge deleting: Fast-forward
.. Sarah Harvey <shh@squareup.com> 1502563865 -0700    merge security: Fast-forward

```

Let's git show that specific commit:

```
~# git show 68490b1f703f667c2dbf9032104a2dcd1039204e

diff --git a/blog/config/secrets.yml b/blog/config/secrets.yml
new file mode 100644
index 0000000..04ec141
--- /dev/null
+++ b/blog/config/secrets.yml
@@ -0,0 +1,24 @@
+# Be sure to restart your server when you modify this file.
+

+flag: flag-a89a24c836bde785292b908a25b9241d
+
+# Do not keep production secrets in the repository,
+# instead read values from the environment.
+production:
+ secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

flag is: flag-a89a24c836bde785292b908a25b9241d

