# Voters

Coming soon...

Here's the goal, I want to continue to require `ROLE_ADMIN_ARTICLE` to be able to go to
the new article page, but down here, if you're editing an article, I want you to be,
I want you allow access if you have the `ROLE_ADMIN_ARTICLE` or if you're the author
of this `Article`. So first let's move. The APP `isGranted()` from above the class just
down to the new method. Now, it only applies to this method in temporarily. Our edit
endpoint is totally wide open to the world now, right now, right now we're looking at
`article_id` 20. It's actually moved back here and once again, look at all the
articles, `article_id` 20. Its `author_id` is 18, so let's run 
```terminal
bin/console doctrine:query:sql 'SELECT * FROM user WHERE id = 18'
```
and okay, cool. This is `spacebar4@example.com`. So let's log in as that user and I go back, 
put a login `spacebar4@example.com`, password `engage` and perfect. We still have access but anyone has
access, but we are at least logged in as the correct user. So the simplest way to do
our logic is just a hard coded in the controller and it's really simple. 
`if ($article->getAuthor() !== $this->getUser())` so we're going to check to see if they 
shouldn't have access so they're not the author and they don't have the `ROLE_ADMIN_ARTICLE` 
role we wanted deny access. Now if you want to just check `true` or `false`, 
if a `User` has a `role`, you can say,

`$this->isGranted()`, I will pass that `ROLE_ADMIN_ARTICLE` so they're not the author and
they don't have the `ROLE_ADMIN_ARTICLE` Role. Then we're going to 
`throw this->createAccessDeniedException('No access!')`. We haven't
seen this method yet because before we were using before we were using 
`$this->denyAccessUnlessGranted()` which did the grant to check and throw the exception for
us. So this is just a little low lower level code when you check is granted, get back
`true` or `false` and then you deny access. By throwing this exception. This message here
is just shown to developers. So for your fresh, we totally get access because we are
the owner of that article, but this sucks. I don't want this logic and my controller.
Why? Well, what if I need this logic somewhere else than I need to duplicate this or
what if I need this inside of twig because I want to show whether or not we show an
edit button for an article. It wouldn't really be a bummer to start having all of
this logic inside of our template. Fortunately, there's a much. There's a wonderful
system in Symfony to handle this called the voter system.

Here's how it works. I'm not clear this whole thing out and I'm going to do something
really strange. I'm going to say, `if (!$this->isGranted('MANAGE', $article))`, so 
I'm using the same functions before, but instead of passing a role, I'm just going to 
invent a string here, `MANAGE` and then he didn't know he could do this on a past. 
The `Article` has the second argument. We'll try this right now. We actually get no access. 
Let me explain what's happening. Whenever you call is granted or denied access and less granted,
those all, all of those methods ultimately do the same thing. It calls, it executes
what's called Symfonys voter system. Basically it takes the string, the string `MANAGE`
here or or more use to `ROLE_ADMIN_ARTICLE` and it asks each voter, do you know how to
decide whether or not the current user has `ROLE_ADMIN_ARTICLE`? Now, in reality, in
the core of Symfony, there are only two voters by default. One voters called roll
voter, and if you pass anything that starts with role_the role voter says, ah, yes, I
know how to determine whether the user has that role or not,

and of course it just uses the roles and the user and it returns `true` or `false` based
on whether or not that user has that role. The other voter, the only other vote of
that's in the system by default is another voter that responds to anything that
starts with `IS_AUTHENTICATED`. So when we say when we use `IS_AUTHENTICATED_FULLY`, the
other voters says, oh, I know how to respond to that and it returns `true` or `false`
based on whichever one of these strings we passed in there.

The really cool thing is we can add our own accustomed voters right now and we call
it, is grants with manage an article. Both of those voters say, we don't understand
what this is, so they both don't vote, and when nobody votes, you get access denied
by default. So all we need to do is introduce a new voter that understands how to
handle the string `MANAGE` and an `Article` object. By the way, I'm using the word `MANAGE`
here, I just made that up. If you need the different permissions for edit and show
and delete, you can make different ones for all of these `EDIT`, `SHOW`, `DELETE`, and you
could make your voter able to handle all of those. You'll see what I mean in a
second. So to create a voter where you want to cheat and run 
```terminal
bin/console make:voter
```
will call one, call the `ArticleVoter`. I usually have one voter per object that I
need to decide access for it. All right, let's go check out that new class

`src/Security/Voter/ArticleVoter.php`. Alright, cool. So there's only two methods
inside of here. Here's how it works. Whenever anybody in the system call calls the
security system, the `supports` method will be called. It's our job to decide whether
or not this voter knows how to vote. In this particular situation. The `$attribute` here
is going to be the `string` passed and these `$subject` here is going to be the second
argument that's passed. If there is anything up until now, we haven't been passing
anything for that second argument. So the example here is actually a pretty good.
We're going to say that we understand how to handle things if the `$attribute` is
`MANAGE` and if the `$subject` is an instance of `Article`. If those two things are true,
we know how to work. If `return false` from supports, nothing happens. Our `Article`, our
voter doesn't vote in some other voter handles it, but if we `return true`, then
Symfony calls.

Our vote on attribute method in our job here is very simple to `return true` if the
users should have access and to `return false` otherwise passes us the same attributes
during the subject and also the token, which is a little lower level here, but you
can see the token is actually used to get access to the `User` object. So first I'm
going to cheat a little bit on the very top of this. I'm going to add `/** @var Article $subject */`
and I'm going to say that the subject is an `Article` I'm doing here is because we have
our supports checkup here. We know that at this point subject will definitely be an
`Article` objects, so I'm just adding a little bit of documentation here for php storm
so that it knows that

right down here it's very common to have a vote or handle multiple conditions. We
don't need that right now, but I'll stick with the switch case statement and of
course our only case is `MANAGE`. All right, so the first thing I want to do is figure
out if the. If this current `User` is the `author` of the `Article`, we should definitely
have access. We'll say `if ($subject->getAuthor() == $user)` than `return true;`
This is the author. Alright. The other case we want to check is whether or not the user has a
`ROLE_ARTICLE_ADMIN`. Now the problem is that in we know how to
check if somebody has a role in a controller, we can choose `$this->isGranted` thing and
he goes back into the voter system, but how can we check that from inside of a voter?

Yeah,

or some other service. The answer is with the `Security` service, which we actually saw
earlier, had a `__constructor`

with a new `$security` argument. The one from the Symfony components, I'll hit enter to
initialize to create that property and set it. You remember when you used the
security service earlier in markdown helper, you can see all the way over here. It's
our last argument and we used it because it gives us access to the current `User`
object. Well, actually there's one other thing that the `Security` class can do hold
command or control, click into it. It has good user on it, but it also has in his
grandson method on it. The security class `Service` is the key to get into `User` or
checking if the user has access. That's actually what the `isGranted` method in the
controller uses behind the scenes. So anyway, down here it's very simple. Now we can
just say `if ($this->security->isGranted('ROLE_ADMIN_ARTICLE'))` then `return true;`
and down here instead of the break, I'm actually, I'm `return false`. Those two conditions
aren't met. We definitely do not have access. So let's try this. Move over, refresh,
and we have access because we are logged in as the current user, our voter gets hit,
life is good and if you want to comment out the author check real quick and refresh,
you'll see we get access denied. That's awesome.

Now one last little thing I'm going to show when we point out is that in our 
`ArticleAdminController`, we have this. We can actually shorten this instead of the whole if
statement. We can use the same, `$this->denyAccessUnlessGranted('MANAGE', $article)` that we used
earlier, past that manage pass it the article object and we're good. That's going to
do the exact same check as before. Refresh, access granted, but wait, we can get even
fancier everywhere else. So far we've been using the annotation. The only problem is
now that we need to pass this in object and actually we can do this with annotations
at `isGranted`. We'll pass it the `MANAGE` key and then if you need a pass, the second
argument too `isGranted`, you can do that by saying `subject="article"`. Now, the reason
this works when you say `subject="article"`, article can be um, any argument that you
have, your controller can be in a, a subject up here. So this will be the equal 70.
We'll pass that argument to it. So now we can get rid of everything, go back,
refresh,

access granted. And just because nothing should work on the first, try a comment on
my author check again. And yes, we get access denied by controller annotation. That
is sweet. Alright guys, that's it. I mean, we have killed on this. We have a great
authentication system which allows for login form authentication, API authentication.
We have a rich dynamic roles system. We have a voter system so we can do access
controls. Oh, I love security. I hope you guys are feeling super empowered to make
your simple, complex, whatever crazy authentication system you have. As always, if
you have questions, um, we're having to talk down in the comments section. All right
guys, go out and build something amazing. See you guys next time.