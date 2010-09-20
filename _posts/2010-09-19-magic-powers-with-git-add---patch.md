---
layout: post
title: Magic powers with `git add --patch`
---

There are many hidden gems in Git. Ever so often I bump into one, either by reading blogs, forums or man pages. Often you just read about this cool feature and marvel at the powers your tools got. I must say tho, I rarely use these fancy, yet clever, features.

There us another class of hidden gems. It's the "I wonder if Git can do this?" kind of gems. Didn't you wish you could work on some features seperated from the others? (branches) Didn't you wish you could see the changes you made? (diff) Didn't you wish you could reorder commits? (rebase)

All of these are those features we realize we need, and happily find in the toolbox Git is. `git add --patch` is the "I wonder if I could commit or add only part of a file?"

I use this feature all the time. When making a commit, I want it to be simple and clean. I literally _craft_ my commits. The commmit message and the changes should be about one feature, fix or change, nothing more nothing less.

## Diving in

Say you have a Git repository where you have made several changes to a file. You might have cleaned up some whitespace, fixed some bugs or made some other changes.

In the end, the diff looks like this:

{% highlight diff%}
diff --git a/lib.coffee b/lib.coffee
index c53fdb4..ec5e1a8 100644
--- a/lib.coffee
+++ b/lib.coffee
@@ -3,10 +3,13 @@
 square = (x) ->
   x * x
 
-cube = (x) ->
+cube   = (x) ->
   x * x * x
 
 # Helpers
 
 testEquality = (a, b) ->
   a is b
+
+testInEquality = (a, b) ->
+  a isnt b
{% endhighlight %}

As we can see, there are both changes to whitespace and changes addition of features! What a mess! Luckily, we got `git add --patch`!

Type the command into the console in the Git repository and watch what happens!

{% highlight diff %}
$ git add --patch 
diff --git a/lib.coffee b/lib.coffee
index c53fdb4..ec5e1a8 100644
--- a/lib.coffee
+++ b/lib.coffee
@@ -3,10 +3,13 @@
 square = (x) ->
   x * x
 
-cube = (x) ->
+cube   = (x) ->
   x * x * x
 
 # Helpers
 
 testEquality = (a, b) ->
   a is b
+
+testInEquality = (a, b) ->
+  a isnt b
Stage this hunk [y,n,q,a,d,/,s,e,?]? 
{% endhighlight%}

It's almost the same as the diff, only this time we are asked what to stage. Let's look at the explanation of the different answers that I'm going to to about:

* "**y**es" means that the hunk (the displayed diff) will be added to the index (it will be added to the commit)
* "**n**o" means that the hunk is not added to the index.
* "**s**plit" means that the hunk is split into smaller hunks, so you can drill down to the part you want.
* "**q**" means that the current and all remaining hunks won't be added to the index.
* "**?**" prints all commands with their explanations.

So looking at the above hunk, it's obvious that we want to **s**plit it to get the whitespace change. Pressing `s` and enter gives the following response:

{% highlight diff %}
Split into 2 hunks.
@@ -3,10 +3,10 @@
 square = (x) ->
   x * x
 
-cube = (x) ->
+cube   = (x) ->
   x * x * x
 
 # Helpers
 
 testEquality = (a, b) ->
   a is b
Stage this hunk [y,n,q,a,d,/,j,J,g,e,?]? 
{% endhighlight %}

No the hunk has been split into two hunks, with this hunk containing only the change we want. We enter `y` because we want to add it, and then `q` because we are done adding. Now we can check that the change is indeed added: 

{% highlight diff %}
$ git diff
diff --git a/lib.coffee b/lib.coffee
index 8edaa0c..ec5e1a8 100644
--- a/lib.coffee
+++ b/lib.coffee
@@ -10,3 +10,6 @@ cube   = (x) ->
 
 testEquality = (a, b) ->
   a is b
+
+testInEquality = (a, b) ->
+  a isnt b
{% endhighlight %}

Now you can add other files, either with `git add` or with `git add --patch`, and finally commit the changes (Just remember to not use `git commit -a`, it will spoil the point of patchin).

When you have commited, you can add the other changes (remember, Git only commits what you add the index with `git add`).

## The magic trick of editing hunks

Imagine that you now keep working, you end up with a diff like the following.

{% highlight diff %}
diff --git a/lib.coffee b/lib.coffee
index ec5e1a8..8353409 100644
--- a/lib.coffee
+++ b/lib.coffee
@@ -8,8 +8,8 @@ cube   = (x) ->
 
 # Helpers
 
-testEquality = (a, b) ->
-  a is b
+testEquality   = (x, y) ->
+  x is y
 
-testInEquality = (a, b) ->
-  a isnt b
+testInEquality = (x, y) ->
+  x isnt y
{% endhighlight %}

As you can see, there are again changes to various unrelated thing, this time tho, we got a problem: There are unrelated changes to the same line!
Even in this situation patching can split up the changes into the relevant parts.
We need to use the 'e' command. We start of by splitting down the hunk, and when only the changes with the "conflict" is in the hunk, we enter 'e'.
Your default editor will now open with the following contents.

{% highlight diff %}
# Manual hunk edit mode -- see bottom for a quick guide (not included here)
@@ -8,6 +8,6 @@

 # Helpers

-testEquality = (a, b) ->
-  a is b
+testEquality   = (x, y) ->
+  x is y

{% endhighlight %}

Along the bottom of your editor there will be a quick guide, but I'm going to explain all we need to know.

You can imagine this as a diff that you can edit. As long as Git understands the diff (it apllies cleanly), Git will accept it and add it to the index.
It behaves just like a normal diff. Lines with '-' are replaced with lines with '+' or nothing.
Let's try and make or change, I'll break it down to small bites. We start by removing all lines we wont need, and we'll start with the whitespace changes.
{% highlight diff %}
-testEquality = (a, b) ->
-  a is b
+testEquality   = (x, y) ->
+  x is y
{% endhighlight %}

As we can see, we wont need line 2 or 4, we can remove them.

{% highlight diff %}
-testEquality = (a, b) ->
+testEquality   = (x, y) ->
{% endhighlight %}

Now, with this in place, we can look at these lines where multiple changes are made. Go to the line, and edit the `(x, y)` to `(a, b)` again.

{% highlight diff %}
-testEquality = (a, b) ->
+testEquality   = (a, b) ->
{% endhighlight %}

Can it be true? _Yes_. When making these changes, all of the diff can be changed, and here we just remove the inline changes to reflect the way we want them.
Git is called the stupid content tracker, and we have to do one final change before the edit is done. We need to add the `a is b` line at the end of the edit, to tell git something about how to apply the patch. If we don't Git wont apply the edit.

In the end, the edited file should look like this.

{% highlight diff %}
# Manual hunk edit mode -- see bottom for a quick guide (not included here)
@@ -8,6 +8,6 @@

 # Helpers

-testEquality = (a, b) ->
+testEquality   = (a, b) ->
   a is b

{% endhighlight %}

Saving the file and closing the editor will make Git go to the next hunk in the diff. If the edit didn't apply, Git would ask us if we wanted to edit again or discard the edit.

Since the next hunk in my example is irrelevant, we can just enter 'q' and commit the changes. Finally, to check that everything is as we want, we can diff the changes we didn't add.

{% highlight diff %}
diff --git a/lib.coffee b/lib.coffee
index 73a5269..8353409 100644
--- a/lib.coffee
+++ b/lib.coffee
@@ -8,8 +8,8 @@ cube   = (x) ->
 
 # Helpers
 
-testEquality   = (a, b) ->
-  a is b
+testEquality   = (x, y) ->
+  x is y
 
-testInEquality = (a, b) ->
-  a isnt b
+testInEquality = (x, y) ->
+  x isnt y
{% endhighlight %}

As we can see, the changes are still intact, allowing us to commit the changes to the code.

## Conclusion

So we learned about `git add --patch` a command that allows us to split up changes, so they can be commited individually. The ability to add only a part of a file is extremely useful when crafting meaningful commits.
To sum up the commands:

   git add --patch
   git add -p
   
When working with hunks we used the following commands.

* y - add hunk
* n - do not add hunk
* q - end the patch
* s - split up hunk into smaller hunks
* e - edit the current hunk

There are few other useful commands.

* j - leave this hunk undecided, see next undecided hunk
* k - leave this hunk undecided, see previous undecided hunk
* / - search for a unk using a regex
* ? - display help messages
