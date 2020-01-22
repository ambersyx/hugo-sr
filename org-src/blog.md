---
author:
- Shreyas Ragavan
hugo_auto_set_lastmod: nil
hugo_base_dir: '\~/hugo-sr/'
hugo_section: post
hugo_weight: auto
---

This is the Org source for all my blog articles

Emacs [[\@Emacs]{.smallcaps}]{.tag tag-name="@Emacs"} [[emacs]{.smallcaps}]{.tag tag-name="emacs"} {#emacs}
==================================================================================================

[DONE]{.done .DONE} Literate Org-mode configuration for Emacs is liberating [[Org\_mode]{.smallcaps}]{.tag tag-name="Org_mode"} [[lisp]{.smallcaps}]{.tag tag-name="lisp"} {#literate-org-mode-configuration-for-emacs-is-liberating}
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------

TLDR: [Check out the Docs section for my Emacs config in
Org-mode](https://shrysr.github.io/docs/sr-config)

> The literate programming paradigm, as conceived by Donald Knuth,
> represents a move away from writing programs in the manner and order
> imposed by the computer, and instead enables programmers to develop
> programs in the order demanded by the logic and flow of their
> thoughts. Literate programs are written as an uninterrupted exposition
> of logic in an ordinary human language, much like the text of an
> essay, in which macros are included to hide abstractions and
> traditional source code.
>
> [Wikipedia article on Literate
> Programming](https://en.wikipedia.org/wiki/Literate_programming)

I had graduated to using an Org-mode based configuration with vanilla
Emacs, until discovering Scimax a few years ago. At this point, it
seemed easier to switch back to using elisp script files in multiple
files which were loaded in the desired / necessary order. The plan was
to use a file for each major \'concept\', for example one file each for
hydras, Org-mode, mu4e, and so on.

While it is not difficult to manage multiple script files with the
projectile package, it does become cumbersome and inelegant to record
notes and thoughts in the comment form along with code. Over time, it
also becomes difficult to decide the placement of multi-package
functions and snippets. As my configuration has evolved - I\'ve felt an
increasing need to shift back to a literate configuration using Org for
Emacs, and also separate the personal parts of my configuration to
enable sharing on Github.

Using a literate configuration enables a live documentary of my Emacs
configuration and also adding meaningful notes and snippets which are
directly or indirectly related to configuring Emacs. For example, it is
important to have IPython and Jupyter installed for Scimax to work
correctly, and I can include notes and working scripts for the same.

There are discussions on Emacs init time increasing by using a tangled
org file. However, this is atleast partially remedied by including a
function to tangle the config file whenever it is saved, and there are
other methods [like the one described by Holger
Schurig](http://www.holgerschurig.de/en/emacs-efficiently-untangling-elisp/),
which I intend to try out soon. Personally, I have not found any degrade
in Emacs init time via Scimax.

[DONE]{.done .DONE} Leverage recorded macros to learn `elisp` and hack together workflows in Emacs [[lisp]{.smallcaps}]{.tag tag-name="lisp"} [[Productivity]{.smallcaps}]{.tag tag-name="Productivity"} {#leverage-recorded-macros-to-learn-elisp-and-hack-together-workflows-in-emacs}
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The primary power of Emacs is that you can create customised workflows
to suit your needs. However, `lisp` is probably not a language that many
learn as a typical requirement in the academic systems, perhaps even for
a software engineer.

How would one then start customisting Emacs? One way would be to hunt
for snippets from forums like reddit and stack overflow, and customise
them.

Another easy way to learn a programming language, especially one that is
intrinsic to a software is to record macros and edit these macros. Emacs
is no different in this regard, and in fact makes it easy being a
self-documenting text editor.

[The elmacro package](https://github.com/Silex/elmacro) reduces some of
the burden. The recorded macro does require a subsequent clean-up to be
useful, which is still easier than coding lisp from scratch. In any
case, exploring the recorded code will eventuall lead towards
proficiency in writing lisp.

[This blog
post](https://emacsnotes.wordpress.com/2018/11/15/elmacro-write-emacs-lisp-snippet-even-when-you-arent-a-programmer/)
provides a more detailed introduction, including creating a menu entry
for elmacro. As highlighted by the blog, some commands do not register
in Emacs, since external packages handle those functions.

For example, I have 3 main repositories where I commit my work. This is
a frequent, repetitive process that is often done till (and at) the last
minute.

These are snippets that were developed leveraging elmacro:

``` {.commonlisp org-language="lisp"}
;; Maximise current frame, open scimax user directory,
;; call magit, switch window and open the scimax directory
;; Scimax magit status and dired
(defun sr/windows-magit-scimax ()
  (interactive)
  (ace-delete-other-windows)
  (dired "~/scimax/user/")
  (switch-window-then-split-right nil)
  (magit-status "~/scimax/")
  (switch-window)
  (split-window-vertically)
  (dired-up-directory)
  (windmove-right)
  )

;; Maximise current frame, open org directory, call magit
;; my_org magit status
(defun sr/windows-magit-org ()
  (interactive)
  (ace-delete-other-windows)
  (magit-status "~/my_org/")
  )

;; Maximise current frame, call magit for my_projects directory
;; split buffer and call dired in case I need to navigate to a particular directory.
;; the latter can also be done via magit itself if desired.
(defun sr/windows-magit-projects ()
  (interactive)
  (ace-delete-other-windows)
  (switch-window-then-split-right nil)
  (magit-status "~/my_projects/")
  (switch-window)
  (dired "~/my_projects/")
  (switch-window)
  )
```

Another more complicated example, is using projectile to switch to a
project, call a particular file in the project and then split the buffer
and open the tasks of that particular project with a narrowed view.

I capture each project\'s tasks and notes separately in an org file
[using org-projectile](/post/8f702ce2-8bb7-40a3-b44b-a47222c02909/).
This is useful especially for coding projects so that the code is better
separated from notes and yet linked.

``` {.commonlisp org-language="lisp"}
;; This is to rapidly switch between projects and have a similar window configuration,
;; i.e. a main file, and a narrowed view of the tasks heading.

(defun sr/windows-projects ()
  (interactive)
  (ace-delete-other-windows)
  (switch-window-then-split-right nil)
  (projectile-switch-project)
  (switch-window)
  (find-file "~/my_org/project-tasks.org")
  (widen)
  (helm-org-rifle-current-buffer)
  (org-narrow-to-subtree)
  (outline-show-children)
  )
```

These are not perfect. For example, I\'d rather have to select the
project name only once and have that feed into `helm-org-rifle`. These
are topics of future exploration.

What then remained was being able call these functions with a few
keypresses. Hydras enable this.

``` {.commonlisp org-language="lisp"}

(defhydra sr/process-window-keys ()
  "
Key^^   ^Workflow^
--------------------
o       org magit
s       scimax magit
p       projects magit
w       select project and set window config
SPC     exit
"
  ("o" sr/windows-magit-org )
  ("p" sr/windows-magit-projects )
  ("s" sr/windows-magit-scimax )
  ("w" sr/windows-projects)
  ("SPC" nil)
  )

(global-set-key (kbd "<f8> m") 'sr/process-window-keys/body)
```

With the above in place, now all I have to do is call the menu to choose
the desired function by typing `F8` `m` and then type `o` or `p` and so
on. The hydra exits with `Space`, which makes it easy to switch to
another project in case there is nothing to commit in the current
choice.

Though simple and in many ways primitive - these functions have still
saved me a lot of repetitive acrobatics on my keyboard and I enjoy using
Them.

[DONE]{.done .DONE} Why bother with Emacs and workflows? :Productivity:yasnippet:Emacs {#why-bother-with-emacs-and-workflows-productivityyasnippetemacs}
--------------------------------------------------------------------------------------

I\'ve written [several posts](https://shrysr.github.io/tags/emacs/) on
different ways and tools available to aid productivity, and probably a
lot about Emacs. My background is in computational physics, and not in
programming, and yet Emacs has been an indispensable driver of my daily
workflow for the past 3 years.

The fact is that knowing Emacs (or Vim), or having a custom
configuration is [not a wildly marketable
skill](https://www.reddit.com/r/emacs/comments/9ghpb4/was_anyone_ever_impressed_by_your_emacs_skills/),
nor is it mandatory to achieve spectacular results. An Emacs
configuration suits personal workflows and style, which may be
borderline peculiar to another person. Such a dependence on customised
tools would also drastically reduces your speed while using a new IDE or
text editor.

So : why add Emacs to the ever-growing to-do list? The question is more
pertinent considering that mastery of a \'text editor\' is not something
you can freely talk about and frequently expect empathetic responses or
even a spark like connection. Emacs would be considered by many to be an
esoteric and archaic software with a steep learning curve that is not
beginner friendly.

However .....

[This article](https://blog.fugue.co/2015-11-11-guide-to-emacs.html)
elucidates many points where Emacs can help PHB\'s (Pointy Haired Boss).
The internet abounds with
[several](https://news.ycombinator.com/item?id=11386590)
[examples](https://news.ycombinator.com/item?id=6094610) on how org-mode
and Emacs have changed lives for the better. Here is another [cool
article by Howard
Abrams](http://www.howardism.org/Technical/Emacs/new-window-manager.html)
on using Emacs as his (only) window manager, in place of a desktop
environment.

Watching an experienced person handle his tools emphasises the potential
art form behind it, especially when compared to the bumbling of an
amateur. Yes, the amateur may get the job done given enough time, and
depending on his capabilities - even match the experienced
professional\'s output (eventually).

However, as experience is gained, the workflows and steps to achieve an
optimal result become more lucid. I\'ve experienced an exponentially
increasing and compelling need to implement specific preferences to
achieve the required optimized results faster and with fewer mistakes.

It is therefore obvious that the workflow and tools used must allow the
provision to evolve, customise and automate. This is particularly true
with respect to the world of data science and programming. I don\'t
think there is anything better than Emacs with respect to customisation.

Imagine the following:

-   having a combination of scripts or snippets in different languages
    to jumpstart a project, which is available with a few keypresses?
    (Yasnippet)[^1]
-   Maintaining a blog with a single document, with articles updated
    automatically on a status change. (ox-hugo)
-   working with multiple R environments in a single document.
    (Org-babel, ESS)[^2]
-   Different Window configurations and processes for different projects
    that can be called with a few keypresses (hint : keyboard macros)
-   An integrated git porcelain that can actually help you learn git so
    much faster (magit)
-   Intimately integrating email with tasks, projects, documentation and
    workflows (mu4e, Org-mode)
-   A customised text editor available right in your terminal (Use
    Emacsclient launched off a daemon within a terminal)
-   Not requiring to use the mouse for navigation![^3]

Now : imagine the consolidated effect of having all the above at your
disposal, in a reasonably streamlined state. Then, considering the
cumulative effect over multiple projects! The above is just a shallow
overview of the possibilities with Emacs.

Investing in learning Emacs, has the serious potential to spawn
exponential results in the long run.

[DONE]{.done .DONE} Rapidly accessing cheatsheets to learn data science with Emacs [[DataScience]{.smallcaps}]{.tag tag-name="DataScience"} [[R]{.smallcaps}]{.tag tag-name="R"} [[emacs]{.smallcaps}]{.tag tag-name="emacs"} {#rapidly-accessing-cheatsheets-to-learn-data-science-with-emacs}
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[Matt Dancho\'s course
DSB-101-R](https://university.business-science.io/p/ds4b-101-r-business-analysis-r)
is an awesome course to step into ROI driven business analytics fueled
by Data Science. In this course, among many other things - he teaches
methods to understand and use cheatsheets to gain rapid *level-ups*,
especially to find information connecting various packages and functions
and workflows. I have been hooked to this approach and needed a way to
quickly refer to the different cheatsheets as needed.

[Favio Vazquez\'s ds-cheatsheets
repo](https://github.com/FavioVazquez/ds-cheatsheets), akin to the One
Ring to Rule them All (with respect to Cheatsheets of course), combined
with Emacs ([Projectile](https://github.com/bbatsov/projectile) +
[Helm](https://github.com/emacs-helm/helm) packages) make it almost a
breeze to find a specific cheatsheet quickly, by just typing in a few
words. [^4]

The built-in Hydras in [Scimax](https://github.com/jkitchin/scimax) make
it very easy to do the above with a few key presses. All I do is `F12`
\>\> p \>\> ww, start typing in \"ds-\" and choose the project and then
start typing in the name of the PDF file I\'m looking for. Check out the
animation below.

![](~/hugo-sr/static/img/Emacs-projectile-cheatsheet.gif)

The above concept applies to switching to any file in any git based
project that is added to Projectile\'s lists.

The next aspect to consider was switching between maximized buffer of
the opened cheatsheet PDF and the current code buffer. As it goes in
Emacs, \"there\'s probably a package for that..\" ! My solution was to
use one of the various frame/window configuration packages in Emacs to
save the position and orientation of the buffers and rapidly switch
between the maximised PDF frame and the split code and interpreter
frames.

Facilitating the above was in fact already available in Scimax, where a
frame or window configuration can be saved into a register that is valid
for that session. Persistent saving of window configuration across
sessions (i.e Emacs restarts) is a little more complex, but it is still
possible with some tweaking. Winner-mode is also an interesting option
to switch rapidly between window configurations.

[DONE]{.done .DONE} Archaic text based email clients rock! [[emacs]{.smallcaps}]{.tag tag-name="emacs"} {#archaic-text-based-email-clients-rock}
-------------------------------------------------------------------------------------------------------

This [dev.to blog
post](https://dev.to/myterminal/how-i-unified-my-email-accounts-in-2019-1pji)
inspired me to complete this languishing draft of my current email
setup, and the benefits I\'ve gained from using a text based email
client in Emacs.

Hope you find it entertaining. In any case, the links and reference
section will certainly prove useful.

### TLDR - for the busy folks

1.  Goals:

    -   Unification of email accounts while preserving separate
        individual components.
    -   Local backup of email.
    -   Potential to extend system to a personal server
    -   Email access from Emacs !
    -   Hopefully improve overall productivity with reduced context
        switching.

2.  Summary:

    1.  Started with 2 Gmail accounts and 1 MSN account.
    2.  Switched to a paid account with Fastmail.
    3.  Used Fastmail\'s tools to transfer email from both Gmail and MSN
        accounts.
    4.  Setup forwarding for all new emails from to Fastmail.
    5.  Decided between retaining copies of emails in Gmail/MSN or
        deleting them once forwarded.
    6.  Used customised settings in mu4e to manage Email from within
        Emacs.
    7.  Occasionally rely on web browser / iOS app. Fastmail\'s
        interface is clean and very fast.
    8.  Goals Achieved !! Live with the quirks and enjoy the perks.

    Look at the [Links and
    References](id:6B67FAC1-7F24-47B6-A8CA-7563849EB4A7) section for
    almost all the resources I relied on.

    A portion of my mu4e configuration is available [on my
    website](https://shrysr.github.io/docs/sr-config/#mu4e). The
    personal filters and configuration are placed in an encrypted file.

    My mbsync configuration is posted as a [public
    gist](https://gist.github.com/shrysr/21676fc69d50337d94c5648b9d31f70a).

### Multiple email accounts. Lack of a unified interface.

Some years back, I found that I had 2 Gmail accounts, and an MSN
account. I discarded age old Yahoo and rediffmail accounts which were
luckily not used much (and God knows how many more I made as a kid).

Gmail\'s interface felt just about tolerable, but inconvenient. The idea
of viewing ads tailored to the content of emails had become
disconcerting. Their Inbox app was interesting, but did not work smooth
enough. MSN\'s web interace and apps always felt cumbersome, though
updates over the years, this has improved significantly.

Useful emails could be email digests that contain a wealth of links,
discussions, articles and information. Or perhaps email digests of
product and technology news that are useful to retain as an archive of
reference.

It would be nice to be able to process these links in a systematic
manner, and have them available with a fast search system that is also
integrated with a task management system.

> My solution was to switch to forwarding all my emails to a single
> Fastmail account. It\'s been an excellent experience over 2+
> years.[^5],[^6]

### Creating sync channels via `mbsync`

My mbsync configuration is posted as a [public
gist](https://gist.github.com/shrysr/21676fc69d50337d94c5648b9d31f70a).
It is reasonably self explanatory, and shows how separate channels were
made grouping together folders, by specifying a pattern. This took some
time, but was finally very satisfying to know as a fine grained control
technique.

> I started out using offlineimap. I found mbsync to be significantly
> faster.

### Text based email client! Speed + simplicity

Imagine being engrossed with your code or engineering notebook and the
need for shooting off an urgent brief email arises. What if this could
be done with a few key-presses on an email client, right from the
terminal or the code editor that you are already engrossed in?

How about adding an email as a task in your organiser with a deadline /
planned date?

What if I had the option to setup separate channels of mail transfer,
such that I can sync the inbox or a custom group of folders alone when I
am pressed for bandwidth or space?

Practical solutions will need to cater to a lot more situations.

> The good news is: usually anything you need is possible (or already
> implemented) using Emacs.

I use [mu4e](https://www.djcbsoftware.nl/code/mu/mu4e.html), which uses
the indexer mu as it\'s back-end. There are other popular options like
[notmuch](https://notmuchmail.org/) and [mutt](http://www.mutt.org/). I
have briefly experimented with mutt, which has a fast email search
capability, but has to be coupled with another front-end to be used
within Emacs or elsewhere. The philosophy and system behind notmuch
(leveraging the Gmail tag based approach) differ from mu4e.

Over a few years of using this system, I have found that text and
terminal based email clients offer a speed and integrity that is
extremely pleasing.

### Why mu4e rocks \[for me\] - the perks

The ability to create custom search filters that can be accessed with
easy shortcuts. An example to demonstrate

``` {.commonlisp org-language="emacs-lisp"}
(setq mu4e-bookmarks
      `( ,(make-mu4e-bookmark
       :name  "Unread messages"
       :query "flag:unread AND NOT flag:trashed"
       :key ?u)
     ,(make-mu4e-bookmark
       :name "Today's messages"
       :query "date:today..now"
       :key ?t)
     ,(make-mu4e-bookmark
       :name "Last 7 days"
       :query "date:7d..now"
       :key ?w)
     ,(make-mu4e-bookmark
       :name "Messages with images"
       :query "mime:image/*"
       :key ?p)
     ,(make-mu4e-bookmark
       :name "Finance News"
       :query (concat "from:etnotifications@indiatimes.com OR "
              "from:newsletters@valueresearchonline.net"
              "from:value research")
       :key ?f)
     ,(make-mu4e-bookmark
       :name "Science and Technology"
       :query (concat "from:googlealerts-noreply@google.com OR "
              "from:reply@email.engineering360.com OR "
              "from:memagazine@asme.org"
              "from:action@ifttt.com"
              "from:digitaleditions@techbriefs.info")
       :key ?S)
         ))
```

This is how it looks:

![](~/hugo-sr/static/img/mu4e-start.png)

Complete keyboard based control, and using it with Emacs means the
ability to compose email from anywhere and build all kinds of workflows.
Examples:

-   Hit Control+x and m (`C-x m`) in Emacs parlance, and I have a
    compose window open.

-   There are built-in workflows and functions in starter-kits like
    [Scimax](https://github.com/jkitchin/scimax), which enable you to
    email an org-heading or buffer directly into an email, with the
    formatting usually preserved, and as intended.

I often use yasnippet to insert links to standard attachments like my
resume. This essentially means being able to attach files with a 1-2 key
strokes.

While Mu4e may be a programmatic solution with no pleasing GUI - it
allows one to search a large number of emails with glorious ease. This
is particularly more effective on a SSD drive, rather than the
conventional Hard disk.

One has to experience the above to *know* the dramatic impact it makes
in getting closer in speed to your thoughts, using a customisable
system. Emails can be easily captured or added as tasks into [Org
mode](https://orgmode.org/) documents as a part of task and project
management.

Using the mu4e and mbsync, I\'ve devised a \'sane inbox\' which is
bereft of the noise, like annoying digests, social media updates and so
on. The idea was to dedicate focused blocks to rapidly process email,
all within Emacs.

I have tried using Todoist extensively in the past, along with their
integration with Gmail. This approach is a reasonable solution, if one
is open to using different applications.

### Quirks

`mu4e` is a text based email interface. It can be set such that the
rendered `HTML` is displayed in the mu4e-view buffer for each email,
which enables graphics and pictures (if any). However, the render is not
perfect at all times. The HTML parsing engine can be specified. Thus,
heavy `HTML` emails are unlikely to render correctly, to the extent of
being a nuisance.

> Such emails can be viewed in the browser of your choice with merely 2
> key presses, \'a\' and then \'v\', with cursor in the body of the
> email. This could be Firefox, or [w3m](http://w3m.sourceforge.net/) or
> any other browser of your choice.[^7]

Email syncing frequency is set in mu4e. This update process takes a few
seconds, and it is not as seamless as a web app. Notifications for new
email can be configured on the mode line or through pop-ups in Emacs.
However, the experience with working synced emails is good.

### Multiple levels of filters are still necessary.

Situations where I do not have access to Emacs will need me to use the
iOS app or the web interface. Therefore the inbox in the web interface
here cannot be \'insane\'. Therefore a higher level of filters are
implemented in Fastmail itself.

For example all Linked in group and job updates have their own folders.
These folders are all subfolders of the Archive. They never reach the
inbox at all. These emails often remain unread, or if necessary, I can
focus on bunches of them at a time.

> By grouping all such incoming mails into subfolders within the Archive
> folder, I can use a single channel for all the *relatively*
> unimportant mail.

### Takeaways

-   Using an \'archaic\' text based email client (mu4e) has
    significantly boosted the speed with which I can handle my emails
    and focus on tasks. The simple interface and speed enables better
    focus.

-   While there are many articles and plenty of guidance on this topic,
    it takes time and patience to get this working the way you need it
    to. However, once it is setup, it does become rather comfortable to
    use.

-   Context switching is expensive on the brain and dents productivity.

-   Integrating email with time and project management is important.
    mu4e integrates well with Org mode. Beyond tasks, it is also a good
    reference, and I can easily attach notes, summaries etc to these
    emails.

### Links and References

These are the links and references I\'ve used in setting up and
troubleshooting my email setup.

> These could be organized better, and some links may be repeated. All
> put together, these should give you all you need to get hooked up!

> Some of the links have additional comments, and many are tagged with
> dates, as a reference to when I collected the link. Sometimes, this is
> fun to reflect on!

-   [A Complete Guide to Email in Emacs using Mu and
    Mu4e](http://cachestocaches.com/2017/3/complete-guide-email-emacs-using-mu-and-/),
    \<2017-03-08 Wed 10:04\>
-   [Reading IMAP Mail in Emacs on OSX \| Adolfo
    Villafiorita](http://www.ict4g.net/adolfo/notes/2014/12/27/EmacsIMAP.html),
    \<2016-11-27 Sun 08:17\>
-   \[ \] Excellent link talking about mu4e and notifications [Handling
    Email with Emacs --
    malb::blog](https://martinralbrecht.wordpress.com/2016/05/30/handling-email-with-emacs/),
    \<2016-08-01 Mon 18:37\>
-   [Which email client (mu4e, Mutt, notmuch, Gnus) do you use inside
    Emacs, and why? :
    emacs](https://www.reddit.com/r/emacs/comments/3s5fas/which_email_client_mu4e_mutt_notmuch_gnus_do_you/)
    \<2016-05-31 Tue 07:32\>
-   [emacs-fu: introducing mu4e, an e-mail client for
    emacs](http://emacs-fu.blogspot.in/2012/08/introducing-mu4e-for-email.html) -
    Emacs and mu4e stuff \<2016-04-20 Wed 13:02\>
-   [Emacs as email client with offlineimap and mu4e on OS X // KG //
    Hacks. Thoughts.
    Writings.](http://www.kirang.in/2014/11/13/emacs-as-email-client-with-offlineimap-and-mu4e-on-osx/) -
    nice blog related to Emacs and linux \<2016-04-21 Thu 22:44\>
-   [EOS: Mail (Email) Module](http://writequit.org/eos/eos-mail.html) -
    explaining multiple email setup in mu4e \<2016-04-27 Wed 07:56\>
-   [The Ultimate Emailing Agent with Mu4e and Emacs -- Emacs, Arduino,
    Raspberry Pi, Linux and Programming
    etc](http://tech.memoryimprintstudio.com/the-ultimate-emailing-agent-with-mu4e-and-emacs/),
    \<2016-08-17 Wed 13:19\>
-   [Varun B Patil \| EOM a.k.a End of Mail a.k.a Emacs + offlineimap +
    mu4e](http://varunbpatil.github.io/2013/08/19/eom/#.VxXTtM7hXCs) -
    multiple accounts \<2016-04-19 Tue 12:19\>
-   [Master your inbox with mu4e and org-mode \| Pragmatic
    Emacs](http://pragmaticemacs.com/emacs/master-your-inbox-with-mu4e-and-org-mode/)
    \<2016-03-26 Sat 14:56\>
-   notmuch - email setup [My personal mail setup --- Articles ---
    WWWTech](https://wwwtech.de/articles/2016/jul/my-personal-mail-setup)
    \<2017-06-13 Tue 16:09\>
-   [Search-oriented tools for Unix-style mail \| Mark J.
    Nelson](http://www.kmjn.org/notes/unix_style_mail_tools.html),
    \<2017-05-10 Wed 16:29\>
    -   interesting comparison of mu and notmuch, going beyond
        superficial differences, but not too much depth either.
-   [Mutt with multiple accounts, mbsync, notmuch, GPG and sub-minute
    updates \| French to English
    translator](https://lukespear.co.uk/mutt-multiple-accounts-mbsync-notmuch-gpg-and-sub-minute-updates),
    \<2017-04-28 Fri 07:19\>
    -   interesting link, author profile and content available on-line.
-   [Assorted Nerdery - Notmuch of a mail setup Part 2 - notmuch
    and Emacs](https://bostonenginerd.com/posts/notmuch-of-a-mail-setup-part-2-notmuch-and-emacs/),
    \<2017-04-27 Thu 18:41\>
-   Mutt, mu4e and notmuch links
    -   [bash - Send Html page As Email using \"mutt\" - Stack
        Overflow](https://stackoverflow.com/questions/6805783/send-html-page-as-email-using-mutt)
    -   [Reading html email
        with mutt](https://fiasko-nw.net/~thomas/projects/htmail-view.html.en)
    -   [Prefer plain text format over HTML in
        mutt](https://xaizek.github.io/2014-07-22/prefer-plain-text-format-over-html-in-mutt/)
    -   [Using emacs and notmuch as a mail client - Foivos . Zakkak .
        net](http://foivos.zakkak.net/tutorials/using_emacs_and_notmuch_mail_client.html)
    -   [Help with mu4e multiple accounts :
        emacs](https://www.reddit.com/r/emacs/comments/4jqyzu/help_with_mu4e_multiple_accounts/)
    -   [Using Mutt, OfflineIMAP and Notmuch to wrangle your inbox. :
        linux](https://www.reddit.com/r/linux/comments/3kj6v4/using_mutt_offlineimap_and_notmuch_to_wrangle/)
        \<2016-06-16 Thu 15:23\>
    -   [A year with Notmuch mail
        {LWN.net}](https://lwn.net/Articles/705856/) \<2018-04-17 Tue
        01:21\>
-   mu4e specific Links \<2016-04-19 Tue 21:48\>
    -   [Mu4e 0.9.16 user manual: Gmail
        configuration](http://www.djcbsoftware.nl/code/mu/mu4e/Gmail-configuration.html#Gmail-configuration)
    -   [mu4e tutorials - Google
        Search](https://www.google.co.in/search?q=mu4e+tutorials&ie=utf-8&oe=utf-8&gws_rd=cr&ei=4IwVV5jkC8fd0ATZ3q2gDA)
    -   [Tutorial: email in Emacs with mu4e and IMAP+SSL :
        emacs](https://www.reddit.com/r/emacs/comments/3junsg/tutorial_email_in_emacs_with_mu4e_and_imapssl/)
    -   [mu4e tutorials \| Pragmatic
        Emacs](http://pragmaticemacs.com/mu4e-tutorials/)
    -   [Drowning in Email; mu4e to the
        Rescue.](http://www.macs.hw.ac.uk/~rs46/posts/2014-01-13-mu4e-email-client.html)
    -   [Emacs & the obsessive email mongerer \| Moved by Freedom --
        Powered by
        Standards](http://standardsandfreedom.net/index.php/2014/08/28/mu4e/)
    -   [Mu4e + nullmailer - Google
        Groups](https://groups.google.com/forum/#!topic/mu-discuss/NzQmkK4qo7I)
    -   [Leaving Gmail Behind « null
        program](http://nullprogram.com/blog/2013/09/03/)
    -   [view html mails in mu4e - Google
        Search](https://www.google.co.in/search?q=view+html+mails+in+mu4e&ie=utf-8&oe=utf-8&gws_rd=cr&ei=e74VV__iOMPM0ASlsq2ACg)
    -   [Mu4e 0.9.16 user manual: Reading
        messages](http://www.djcbsoftware.nl/code/mu/mu4e/Reading-messages.html)
    -   [In mu4e, is this how your HTML-heavy emails render? :
        emacs](https://www.reddit.com/r/emacs/comments/1xad11/in_mu4e_is_this_how_your_htmlheavy_emails_render/)
    -   [Varun B Patil \| EOM a.k.a End of Mail a.k.a Emacs +
        offlineimap +
        mu4e](http://varunbpatil.github.io/2013/08/19/eom/#.VxXTtM7hXCs)
    -   [Mu4e 0.9.16 user manual: Marking
        messages](http://www.djcbsoftware.nl/code/mu/mu4e/Marking-messages.html#Marking-messages)
    -   [change the date column format in mu4e - Google
        Search](https://www.google.co.in/search?q=change+the+date+column+view+in+mu4e&ie=utf-8&oe=utf-8&gws_rd=cr&ei=TDgWV8zEBIOLuwTXk5uYAw#q=change+the+date+column+format+in+mu4e)
    -   [Mu4e 0.9.16 user manual: HV
        Overview](http://www.djcbsoftware.nl/code/mu/mu4e/HV-Overview.html)
    -   [increase column size in mu4e - Google
        Search](https://www.google.co.in/search?q=increase+column+size+in+mu4e&ie=utf-8&oe=utf-8&gws_rd=cr&ei=ZjsWV7TDLJW3uQT6qZEY)
    -   [Mu4e 0.9.16 user manual: HV Custom
        headers](http://www.djcbsoftware.nl/code/mu/mu4e/HV-Custom-headers.html)
    -   [mu4e-manual-0.9.9.pdf](https://ftp.fau.de/gentoo/distfiles/mu4e-manual-0.9.9.pdf)
    -   [do mu4e folders sync with gmail folders - Google
        Search](https://www.google.co.in/search?q=do+mu4e+folders+sync+with+gmail+?&ie=utf-8&oe=utf-8&gws_rd=cr&ei=7DsWV7-NHIyXuASgtJ44#q=do+mu4e+folders+sync+with+gmail+folders)
    -   [mu4e Send mail with custom SMTP and archive in Gmail \"Sent\"
        folder :
        emacs](https://www.reddit.com/r/emacs/comments/3r8dr3/mu4e_send_mail_with_custom_smtp_and_archive_in/)
    -   [Using mu4e · Brool ](http://www.brool.com/post/using-mu4e/)
    -   [are maildir folders synced back to gmail ? - Google
        Search](https://www.google.co.in/search?q=are+maildir+folders+synced+back+to+gmail+?&ie=utf-8&oe=utf-8&gws_rd=cr&ei=RlwWV5TKKI62uASltLz4Ag)
    -   [Some real use
        cases](http://www.offlineimap.org/doc/use_cases.html)
    -   [About](http://deferred.io/about/)
    -   [Backing up Gmail messages with
        offlineimap](https://bluishcoder.co.nz/2013/04/30/backing_up_gmail_messages_with_offlineimap.html)
    -   [notmuch email versus mu4e - Google
        Search](https://www.google.co.in/search?q=notmuch+email+versus+mu4e&ie=utf-8&oe=utf-8&gws_rd=cr&ei=zmcWV8eVEIqdugTzkIpo)
    -   [Which email client (mu4e, Mutt, notmuch, Gnus) do you use
        inside Emacs, and why? :
        emacs](https://www.reddit.com/r/emacs/comments/3s5fas/which_email_client_mu4e_mutt_notmuch_gnus_do_you/)
    -   [A Followup on Leaving Gmail \|
        Irreal](http://irreal.org/blog/?p=2897)
    -   [Sup?](http://cscorley.github.io/2014/01/19/sup/)
    -   [Mutt + Gmail +
        Offlineimap](https://pbrisbin.com/posts/mutt_gmail_offlineimap/)
    -   [Migrating from offlineimap to mbsync for mu4e \| Pragmatic
        Emacs](http://pragmaticemacs.com/emacs/migrating-from-offlineimap-to-mbsync-for-mu4e/)

[DONE]{.done .DONE} Juggling multiple projects and leveraging org-projectile [[Productivity]{.smallcaps}]{.tag tag-name="Productivity"} [[emacs]{.smallcaps}]{.tag tag-name="emacs"} [[orgmode]{.smallcaps}]{.tag tag-name="orgmode"} {#juggling-multiple-projects-and-leveraging-org-projectile}
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[Scimax](https://github.com/jkitchin/scimax) has a convenient feature of
immediately creating projects (`M-x nb-new`). The location of the
project directory is defined by the setting
`(setq nb-notebook-directory "~/my_projects/")`, which has to be set in
your Emacs config. Once the name of the project is chosen, a Readme.org
buffer is immediately opened and one can start right away. It is an
awesome, friction-free method to get started with a project.

These projects are automatically initialised as git repositories, to
which it is trivial to add a new remote using Magit. Therefore
individual folders and git repos are automatically created for each
project in the specified project directory. This enables the convenient
possibility of keeping the data, folder structures, tasks, notes and
scripts of each project separate.

Different projects can be switched to using `M-x nb-open` and typing in
a few words that denote the title of the project. Choosing a project
automatically provides the option to open the Readme.org files created
earlier. Therefore it would be convenient to include relevant links to
different locations / scripts and etc in the Readme file.

Using the above technique resulted in me creating a huge number of
projects over a period of time. Especially while working on multiple
computers, it is worth inculcating the discipline of adding a remote on
github/bitbucket and regularly pushing to the remote.

The advantage of using a separate repo for each project is the alignment
with the space constraints imposed by the free tier repos on bitbucket
or github. However, it is also useful to have the entire project folder
as a git repo. This can be resolved by adding each project as a
sub-module. In this way, all the projects are available with a single
clone of the project foder, and then specific sub-modules or projects
can be initialized as required. Having separate repos for each project
also enables more streamlined collaboration or publishing of a
particular project, rather than the entire project folder and allowing
separate gitignore lists for each project.Using a single file for all
the projects will also enable adding notes pertaining to the content of
each project, which can be searched before intialising the entire
project repo. Scripts for initializing and commit can also be included
in this file for convenience.

Once the above is done, the
[org-projectile](https://github.com/IvanMalison/org-projectile/blob/master/org-projectile.el)
package can be leveraged to plan the tasks and manage the notes for each
project. It is possible to have all the tasks for a project within a
separate file within each project, or specify a single file as the task
management for all the projects. This file is then appended to the
org-agenda files for tasks to show up in the agenda. As mentioned in the
Readme of the org-projectile package the settings would look like the
following (for a single file pertaining to all the projects):

``` {.commonlisp org-language="lisp"}
;; Setting up org-projectile
(require 'org-projectile)
(setq org-projectile-projects-file
      "~/my_org/project-tasks.org")
(push (org-projectile-project-todo-entry) org-capture-templates)
(setq org-agenda-files (append org-agenda-files (org-projectile-todo-files)))
(global-set-key (kbd "C-c n p") 'org-projectile-project-todo-completing-read)
```

The above snippet adds a TODO capture template activated by the letter
\'p\', and also adds the `project-tasks` file to the agenda files.
Inside a project, it is then possible to capture using `C-cc p` and add
a task which will create a top level heading linked to the project, and
the task or note as a sub-heading.

[DONE]{.done .DONE} Jupyter notebooks to Org source + Tower of Babel [[DataScience]{.smallcaps}]{.tag tag-name="DataScience"} [[Jupyter]{.smallcaps}]{.tag tag-name="Jupyter"} [[Python]{.smallcaps}]{.tag tag-name="Python"} [[orgmode]{.smallcaps}]{.tag tag-name="orgmode"} {#jupyter-notebooks-to-org-source-tower-of-babel}
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

This post provides a simple example demonstrating how a shell script can
be called with appropriate variables from any Org file in Emacs. The
script essentially converts a Jupyter notebook to Org source, and
[Babel](https://orgmode.org/worg/org-contrib/babel/) is leveraged to
call the script with appropriate variables from any Org file. This
[reddit thread](https://news.ycombinator.com/item?id=11296843) and [blog
post](https://lepisma.github.io/2016/11/02/org-babel/) elucidate the
advantages of using Babel and Org mode over Jupyter notebooks.

Directly editing code in a Jupyter notebook in a browser is not an
attractive long term option and is inconvenient even in the short term.
My preference is to have it all in Emacs, leveraging a versatile Org
file where it is easy to encapsulate code in notebooks or projects
within Org-headings. Thus, projects are integrated with the in-built
task management and calendar of Org mode.

However, it may be a frequent necessity to access an external Jupyter
notebook for which there is no Org source.

One solution is to start up a Jupyter server locally, open the file and
then File \>\> save as a markdown file, which can be converted to an Org
file using pandoc. Remarkably, the output code seems similar to the code
blocks used in the R-markdown notebooks, rather than pure markdown
markup. Therefore this markdown export should work fine in RStudio as
well. However, unless the Jupyter server is always running on your
machine, this is a relatively slow, multi-step process.

[This SO
discussion](https://emacs.stackexchange.com/questions/5465/how-to-migrate-markdown-files-to-emacs-org-mode-format)
provided my answer, which is a 2 step script via the versatile
[pandoc](https://pandoc.org/). A workable solution, as a test conversion
revealed. The headings and subheadings and code are converted into Org
markup along with Org source blocks.

``` {.shell}
jupyter nbconvert notebook.ipynb --to markdown
pandoc notebook.md -o notebook.org
```

The next consideration was to have the above script or recipe handy for
converting any Jupyter notebook to an Org file quickly.[^8] For the
script to be referenced and called from any other location, the source
block needs to be defined with a name and the necessary arguments, and
also added into the org-babel library.

In this example the path to the Jupyter notebook, markdown file and
resulting org file are specified as variables or arguments. Note that
the absolute path to any file is required. Save the following in an Org
file, named appropriately, like my-recipes.org

``` {.commonlisp org-language="emacs-lisp"}
#+NAME: jupyter-to-org-current
#+HEADER:  :var path_ipynb="/Users/xxx/Jupyter_notebook"
#+HEADER: :var path_md = "Jupyter_notebook-markdown"
#+HEADER: :var path_org = "Jupyter-notebook-org"
#+BEGIN_SRC sh :results verbatim
cwd=$(pwd)
jupyter nbconvert --to markdown $path_ipynb.ipynb --output $cwd/$path_md.md
pandoc $cwd/$path_md.md -o $cwd/$path_org.org
cp $path_ipynb.ipynb $cwd
ls
```

The `path_ipynb` variable can be changed as required to point to the
Jupyter notebook.[^9]

All such blocks above can be stored in Org files and added to the
Library of Babel (LOB) by including the following in the Emacs init
configuration.

``` {.commonlisp org-language="lisp"}
(org-babel-lob-ingest "/Users/shreyas/my_projects/my-recipes.org")
```

The named shell script source block can now be called from any Org file,
with specified arguments and have the notebook. The script is called
using the `#+CALL` function and using the name and arguments of the
source block above.

``` {.commonlisp org-language="lisp"}
#+CALL: jupyter-to-org-current(path_md="Jup-to-markdown", path_org="Markdown-to-org")
```

Therefore, the snippet above will convert a Jupyter notebook to a
markdown file named `Jup-to-markdown` and then an Org file called
`Markdown-to-org`. If an argument is not specified, the default value of
the paths specified in the original source block will be used.

Of course, the `#+CALL` function used above is also too lengthy to
remember and reproduce without headaches. This is also bound to happen
as the number of such named code snippets increase. One solution (though
not ideal) is to store the `#+CALL` as a snippet using `M-x`
`yas-new-snippet`, and load it when needed using the excellent
`ivy-yasnippet` package (see MELPA), with minimal exertions.

### Further possibilities

It would be nice to improve the options available for modifications on
the fly. Python may be an \'easier\' option to write up for such
activities rather than a shell script. For example, a script with the
working directory being an additional /optional argument could be
considered.

Another desirable factor in the resulting Org file would be iPython
blocks in place of python. As a temporary solution, the python blocks
could be converted to ipython blocks via a search and replace throughout
the document. A lisp macro / source block could run after the above
source block to facilitate the search and replace. [^10]

[DONE]{.done .DONE} Emacs notes: Select paragraph and browse-kill-ring for effective content capture [[lisp]{.smallcaps}]{.tag tag-name="lisp"} [[emacs]{.smallcaps}]{.tag tag-name="emacs"} {#emacs-notes-select-paragraph-and-browse-kill-ring-for-effective-content-capture}
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

I like to have any reading material and my notes side by side[^11]. This
is easily done with Emacs by splitting the buffer vertically
(`C-x 3`)[^12]

For example: Once a link has been opened via w3m, I hit org-capture
(`C-c`) with a preset template that grabs the URL to the article along
with the created date in the properties, with the cursor in position
ready to take notes.

``` {.commonlisp org-language="lisp"}
(setq org-capture-templates
'(("l" "Link + notes" entry (file+headline "~/my_org/link_database.org" ".UL Unfiled Links")
     "** %? %a ")))
```

The snippet above is activated by the command \'l\' and is listed with
the title Link + notes in the agenda. It captures the link of the file
being viewed as the heading and allows further notes to be inserted
below. This is stored into the file `link_database` and under the
specified heading `.UL Unfiled Links`.

It is also possible to capture a highlighted chunk of text to be added
under the heading mentioned above. That would look something like:

``` {.commonlisp org-language="lisp"}
(setq org-capture-templates
    '(("e" "Snippet + Notes" entry ;; 'w' for 'org-protocol'
     (file+headline "~/my_org/link_database.org" ".UL Unfiled Links")
     "*** %a, %T\n %:initial")))
```

Now I have the capture buffer and the viewing content side by side, by
calling `C-c l`. I can browse through the article use the mark-paragraph
function (conveniently set to `M-h`) can be used to select and copy
(`M-w`) entire paragraphs or alternately use `C-spc` to select lines of
interest from the article them to the kill ring. The figure below
depicts how it looks for me:

![](~/hugo-sr/static/img/capture-content-emacs.png)

It is now possible to continue highlighting interesting lines /
paragraphs and copy them, which adds them to the kill-ring. Once the
article is done with, I switch over to the capture buffer and hit `M-x`
browse-kill-ring, which brings up a pop-up buffer with all the items in
the kill-ring[^13]. Once called, I can hit n to move to the next item,
and hit \'i\' to insert the current item at the cursor location. It is
also possible to append / prepend/ edit the item before yanking. All the
available shortcuts can be found using \'?\', while in the
browse-kill-ring buffer.

The above methodology curiously enables me to ensure capturing atleast
some details of interest from an article / source, and also serve as a
quick revision of the read content before filing it away.

One issue with the above workflow is that while reading multiple
articles, there is a chance of mixing up the content being captured from
different articles. This could be solved by using \'x\' in order to pop
items out of the kill ring in the selection process above. However, it
seems excessive to clear the entire kill ring for each article read. On
the other hand, it could promote a focused workflow.

Additional possibilities:

-   To view pdf files side by side and capture notes is via the
    [Interleave package](https://github.com/rudolfochrist/interleave).
-   The org-web-clipper concept outlined
    [here](http://www.bobnewell.net/publish/35years/webclipper.html) is
    also very convenient to rapidly capture entire webpages being
    browsed in w3m.

Further reading:

-   Howard Abrams has [some great
    tips](http://www.howardism.org/Technical/Emacs/capturing-intro.html)
    on customising the org-capture mechanism,
-   [Bernt Hansen\'s comprehensive
    documentation](http://doc.norang.ca/org-mode.html).

[DONE]{.done .DONE} Iosevka - an awesome font for Emacs [[writing]{.smallcaps}]{.tag tag-name="writing"} [[font]{.smallcaps}]{.tag tag-name="font"} [[Linux]{.smallcaps}]{.tag tag-name="Linux"} [[Productivity]{.smallcaps}]{.tag tag-name="Productivity"} [[Emacs]{.smallcaps}]{.tag tag-name="Emacs"} {#iosevka---an-awesome-font-for-emacs}
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Before my foray into Emacs, I purchased applications like
[IAWriter](https://ia.net/writer) (classic)[^14],
[Marked2](http://brettterpstra.com/2017/08/01/long-form-writing-with-marked-2-plus-2-dot-5-11-teaser/),
[Texts](http://www.texts.io/) (cross platform Mac/Windows), and have
also tried almost all the recommended apps for longer form writing. I am
a fan of zen writing apps. In particular the font and environment
provided by IAWriter are conducive to focused writing. There also exist
apps like Hemingway that also help check the quality of your writing.

Zen writing apps are called so because they have a unique combination of
fonts, background color, including line spacing and overall text-width -
all of which enable a streamlined and focused flow of words onto the
screen. Any customisation required towards this end is possible in
Emacs.

The Texts app has some nifty features (besides being cross platform),
but the font and appearance is not as beautiful as IAWriter. Both
IAWriter (classic) and Texts have minimal settings for further
customisation. See the comparison below:

![](~/hugo-sr/static/img/emacs-texts.png)

![](~/hugo-sr/static/img/emacs-iawriter.png)

While everybody\'s style and approach vary, there are many authors who
swear by archaic text editors and tools that enable distraction free
writing. One example is [Tony Ballantyne\'s post on writing
tools](http://tonyballantyne.com/how-to-write/writing-tools/), and
several more examples are available in this [blog
post](http://irreal.org/blog/?p=4651).

The next best thing to a clear retina display on a MacBook Pro, is a
beautiful font face to take you through the day, enhancing the viewing
pleasure and thus the motivation to work longer.

In Emacs,
[writeroom-mode](https://github.com/joostkremers/writeroom-mode) and
Emacs can be customised to mimic IAWriter. In this regard, the font
[Iosevka](https://be5invis.github.io/Iosevka/), is a great font to try.
This [old Emacs
reddit](https://www.reddit.com/r/emacs/comments/5twcka/which_font_do_you_use/)
has many more suggestions. One post described Iosevka as *\"it*
*doesn\'t look like much, but after a few hours it will be difficult to*
*use any other font.\"* This is exactly what happened to me.

There\'s still a lot of tweaking to be done with `writeroom-mode`, but
this is certainly a workable result. My nascent configuration for
writeroom-mode in emacs is as follows (munged off the internet!). It\'s
remarkable how much was achieved with a few lines of code!

``` {.commonlisp org-language="lisp"}
(with-eval-after-load 'writeroom-mode
  (define-key writeroom-mode-map (kbd "C-s-,") #'writeroom-decrease-width)
  (define-key writeroom-mode-map (kbd "C-s-.") #'writeroom-increase-width)
  (define-key writeroom-mode-map (kbd "C-s-=") #'writeroom-adjust-width))

(advice-add 'text-scale-adjust :after
        #'visual-fill-column-adjust)
```

[DONE]{.done .DONE} Searching the awesome-lists on Github [[Productivity]{.smallcaps}]{.tag tag-name="Productivity"} {#searching-the-awesome-lists-on-github}
--------------------------------------------------------------------------------------------------------------------

Discovered the glorious awesome lists today on Github. They are
available through a [simple search on
github](https://github.com/search?utf8=%E2%9C%93&q=awesome+list&type=),
and contain curated lists of resources of all kinds on a multitude of
topics.

As one might expect, there is a lot of common ground between these
lists, including topics and links.

How could one search for a keyword through all these repositories? I
have always wanted search for particular keywords or code snippets in my
Emacs configuration files, or in other files in a particular location.
This is especially to verify if a bit of code or note is already
available, in another location. Something that looks like this ;):

![](~/hugo-sr/static/img/emacs-helm-ag-anim.gif)

An answer had been available in [Howard\'s cool blog
post](http://www.howardism.org/Technical/Emacs/why-emacs.html) on why
one should learn Emacs - in a footnote (!), in which he\'s mentioned
`ack` and `ag` ([the silver
searcher](https://github.com/ggreer/the_silver_searcher)). [^15]. It is
even possible to edit in line with each search.

The silver searcher github page provides clear examples of how it\'s
significantly faster than ack (and similar tools). Further exploration
led me to the [emacs-helm-ag](https://github.com/syohex/emacs-helm-ag)
package, which is a helm interface to [the silver
searcher](https://github.com/ggreer/the_silver_searcher). Implementing
emacs-helm-ag was as simple as adding it to my list of packages, and
adding a basic setup to my helm configuration.[^16]

As of now, I add packages to
[Scimax](https://github.com/jkitchin/scimax) using this bit of code that
I\'ve obviously borrowed from the internet, and this case - I\'m afraid
I did not note the source.

``` {.commonlisp org-language="lisp"}
;; Setting up use packages
;; list the packages you want
(setq package-list '(diminish org-journal google-this ztree org-gcal w3m org-trello org-web-tools ox-hugo auto-indent-mode ob-sql-mode dash org-super-agenda ox-hugo workgroups2 switch-window ess ess-R-data-view interleave deft org-bookmark-heading writeroom-mode evil evil-leader polymode helm-ag))

;;fetch the list of packages available
(unless package-archive-contents
  (package-refresh-contents))

;; install the missing packages
(dolist (package package-list)
  (unless (package-installed-p package)
    (package-install package)))

;; Remember to start helm-ag. As per the Silver searcher github site, the helm-follow-mode-persistent has to be set before calling helm-ag.

(custom-set-variables
 '(helm-follow-mode-persistent t))

(require 'helm-ag)
```

This is how it looks in action \>\> Sweet !!

![](~/hugo-sr/static/img/helm-ag-emacs.png)

[DONE]{.done .DONE} Literate Programming - Emacs, Howard Abrams and Library of Babel [[Productivity]{.smallcaps}]{.tag tag-name="Productivity"} [[emacs]{.smallcaps}]{.tag tag-name="emacs"} {#literate-programming---emacs-howard-abrams-and-library-of-babel}
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

I\'m an admirer of [Howard
Abrams](https://www.linkedin.com/in/howardeabrams/), especially because
his posts and videos show the awesome power of doing things in Emacs,
and the importance of writing clean and logical code. Watching his
videos and reading his posts make me feel like I was born yesterday and
I am just getting started. But more importantly, they also fire up my
imagination regarding the possibilities out there and the potential to
create glorious workflows.

Howard\'s tutorial on [Literate
Programming](Http://www.howardism.org/Technical/Emacs/literate-programming-tutorial.html),
combined with his [Literate Devops with Emacs
video](https://www.youtube.com/watch?v=dljNabciEGg) are among the best
ways to get started with understanding the power of using Org Mode and
Org-Babel to create complex, inter-connected, multi-language programs /
documents / research that are of course well documented (this being one
basic tenet of literate programming). Essentially, Org Mode and
Org-Babel enable a high quality programming environment in a single Org
mode buffer or document. The said environment is significantly more
feature rich compared to Jupyter notebooks, especially being supported
by it\'s foundation in Emacs.

Though I\'ve been using Org files for a while now for all my programming
explorations, I\'ve been bothered about my sub-par workflows. I could
not easily reference other code blocks and snippets and recipes for a
new document or project. It was inefficient and time consuming to locate
the necessary snippet and re-write or re-paste the code in the new
source blocks. I was not making much progress plodding through the vast
documentation of org-babel.

Therefore, I was thrilled to discover the [Library of
Babel](https://orgmode.org/worg/org-contrib/babel/library-of-babel.html)
through Howard\'s tutorial, which can be used to add files to a global
library that is accessible from anywhere! Did I mention that it involves
hitting barely 3 keys, and any number of arguments can be passed to
these source blocks? I\'m not sure such a feature is available with any
other IDE.

In addition, the above tutorial clearly elucidates how different
languages can be combined together, and the video elucidates typical
Devops procedures, which are easily taken care of with appropriate
arguments and headers to the source code blocks. For example, all the
source code blocks could be tangled into appropriately named and located
script files using a single argument. These tutorials tied up bits and
pieces of info in my head from various sources and was invaluable in
enhancing my understanding of using Emacs and Org-Babel

The Library of Babel can be made persistent across sessions by loading a
specified org-file from which the named source code blocks are
automatically read in. It is surprising that the internet does not seem
to contain more references and examples using the Library of Babel.
Perhaps there are some caveats that I am yet to encounter. One question
that arises is whether the Library of Babel is automatically updated
when the source code block is updated.

Data Science [[\@DataScience]{.smallcaps}]{.tag tag-name="@DataScience"} [[DataScience]{.smallcaps}]{.tag tag-name="DataScience"} {#data-science}
=================================================================================================================================

[TODO]{.todo .TODO} tmux and mosh - two excellent tools that any terminal friendly person should be aware of {#tmux-and-mosh---two-excellent-tools-that-any-terminal-friendly-person-should-be-aware-of}
------------------------------------------------------------------------------------------------------------

I wrote recently about getting started with using mosh for interacting
with my VPS. Thankfully, I was directed to such solutions from the
excellent folks in the \#emacs IRC channel.

mosh is short for mobile shell. Essentially what mosh does is create a
server that interfaces with the remote system\'s shell and synchronizes
itself with a mosh server running on your local computer. The benefit of
doing this is that even if your internet is patchy and disconnects - the
mosh sync will make the process significantly smoother. There will be a
clear indication of a disconnect. The prime benefit occurs over spotty
internet connections where one experiences a lot of lag between typing
and seeing the characters appear.

Using mosh is essentially like having a terminal on your remote server
that is always running and you can connect to it.

Now let\'s say I have multiple running processes on my server that I
want to monitor. One thing I want to monitor is the system itself. Enter
htop. Next I would like to tail some log files of perhaps shiny apps or
the NGINX webserver. Perhaps I\'d ever like an RSS ticker :) In fact, I
would like one window open connected to IRC on my server.[^17]

How can one manage all the above? Enter tmux. This stands for terminal
multiplexer. By using tmux you can not only have multiple shells
connected to the same mosh instance, but also configure the window
positioning and keyboard shortcuts

Here\'s a picture of my terminal tab. So this is a single tab open on my
local computer. Note the tabs at the bottom which show you that I have
multiple tabs open. I connect to the mosh server simply by replacing the
usual `ssh` with `mosh`.

[DONE]{.done .DONE} A graphic overview of the \'binary\' with respect to R packages {#a-graphic-overview-of-the-binary-with-respect-to-r-packages}
-----------------------------------------------------------------------------------

Recently there was a question as to what a Binary is, building off a
question [posted on the Rstudio community
forum](https://community.rstudio.com/t/meaning-of-common-message-when-install-a-package-there-are-binary-versions-available-but-the-source-versions-are-later/2431).
I\'ve always found these aspects interesting, and a little hard to keep
track of the connections and flow - So I\'ve made a flowchart that will
help me remember and hopefully explain what is happening to a noob.

In this process, I was able to remember [One of the
first](http://www.tldp.org/HOWTO/Unix-and-Internet-Fundamentals-HOWTO/)
documents I really enjoyed reading when I started learning how to use
Linux. I would recommend that article for anybody starting out. The
document is meant for people with a non-technical background, but I
think it is technical enough.

So: Lets pretend the binary is a capsule to be swallowed by the computer
to gain superpowers :D : The capsule is in a sense off-the-shelf and
made for your System. When the capsule is not available, it has to be
manufactured (compiled) by your machine locally for which it needs
certain tools and dependencies, etc and this varies from package to
package, and possibly between hardware architectures as well and more.
Please feel free to let me know if there are any discrepancies in the
flowchart !

Binaries can be built and maintained if you desire. There should be
people maintaining their own binaries or frozen versions as well. The
question is - who is going to maintain them and how many binaries can
you build.

![](~/hugo-sr/static/img/binaries-source-code-11.jpg)

[DONE]{.done .DONE} Some notes on research-compendium [[R]{.smallcaps}]{.tag tag-name="R"} [[DataScience]{.smallcaps}]{.tag tag-name="DataScience"} {#some-notes-on-research-compendium}
---------------------------------------------------------------------------------------------------------------------------------------------------

These are my notes while studying the research-compendium concept, which
is essentially a bunch of guidelines to produce research that is
\'easily\' reproducible.

The notes are mostly based on <https://peerj.com/preprints/3192/>, which
is recommended as a canonical reading on the concept. Other references
are mentioned throughout the text. These notes were prepared a few weeks
ago during a foray into Docker. They are neither complete not
comprehensive - but will serve as a good refresher of the principle
concepts.

-   [Landing page](https://research-compendium.science/) : contains
    several references explaining research-compendium.
-   Principles
    -   stick with the prevailing conventions of your peers / scholarly
        community
    -   Keep data, methods and outputs separate, but make sure to
        unambiguously express the connections between them. The result
        files should be treated disposable (can be regenerated).
    -   Specify computational environment as clearly as possible.
        Minimally, a text file specifying the version numbers of the
        software and other critical tools being used.
-   R\'s package structure is conducive to organise and share a
    compendium, for any project.
-   Dynamic documents : essentially like org files or Rmarkdown files
    i.e. literate programming. Sweave was originally introduced
    around 2002. However, around 2015 : knittr and rmarkdown made
    substantial progress and are in general more preferred than using
    sweave.
-   Shipping data with the packages
    -   CRAN : generally less than 5MB. A large percentage of the
        packages have some form of data. Data should be included if a
        methods package is being shipped with the analysis.
    -   use the [piggyback](https://github.com/ropensci/piggyback)
        package for attaching large datafiles to github repos.
        -   It is convenient to be able to upload a new dataset to be
            associated with thep package, and this can be accessed with
            `pb_download()`.
    -   \"medium\' sized data files can be attached using
        [arkdb](https://github.com/ropensci/arkdb)
-   Adding a Dockerfile to the compendium
    -   containerit : o2r/containerit
    -   repo to docker : jupyter/repo2docker
    -   Binder : <https://mybinder.org>
    -   Use the [holepunch](https://github.com/karthik/holepunch)
        package to make the setup easier.
-   Summarising the folder structure for R packages esque
    -   Readme file : self-explanatory and should be as detailed as
        possible, and preferably include a graphical connection between
        various components.
    -   `R/` : Script files with resusable functions go here. If roxygen
        is used to generate the documentation, then `man/` dicrectory is
        automatically populated with this.
    -   `analysis/` : analysis scripts and reports. Considering using
        ascending names in the file names to aid clarity and order eg
        001-load.R, 002 -... and so on.
    -   The above does not capture the dependencies. Therefore an .Rmd
        or `Makefile` (or `Makefile.R`) can be included to capture the
        full tree of dependencies. These files control the order of
        execution.
    -   `DESCRIPTION` file in the project root provides formally
        structured, machine and human-readable information on authors /
        project license, software dependenceis and other meta data.
        -   when this file is included, the project becomes an
            installable R package.
    -   `NAMESPACE`: autogenerated file that exports R functions for
        repeated use.
    -   `LICENSE` : specifying conditions for use /reuse
-   \[ \] Drone : CI service that operates on Docker containers. This
    can be used as a check.
-   `Makefiles`
    -   uses the make language.
    -   specifies the relationship between data, the output and the code
        generating the output.
    -   Defines outputs (targets) in terms of inputs (dependencies) and
        the code necessary to produce them (recipes).
    -   Allows rebuilding only the parts that are out of date.
    -   the `remake` package enables write Make like instructions in R.
-   Principles to consider before sharing a research compendium
    -   Licensing, Version control, persistence, metadata : main aspects
        to consider.
    -   Archive a specific commit at a repository that issues persistent
        URL\'s eg DOI which are designed to be more persistent than
        other URL\'s. Refere re3data.org for discipline-specific DOI
        issuing repositories. Using a DOI simplifies citations by
        allowing the transfer of basic metadata to a central registry
        (eg CrossRef and Datacite). Doing this ensures that a publicly
        available snapshot of code exists that can match the results
        published.
    -   CRAN is generally not recommended for research-compendium
        packages, because it is strict about directory structures and
        contents of the R packages. It also has a 5MB limit for package
        data and documentation.
-   Tools and templates
    -   `devtools`
    -   `rrtools` : extends devtools

Reference list

-   <https://ropensci.org/commcalls/2019-07-30/?eType=EmailBlastContent&eId=2d18a2f6-57ef-4d15-8c52-84be5c49e039>
    \| rOpenSci \| Reproducible Research with R
-   <https://github.com/annakrystalli/rrtools-repro-research> \|
    annakrystalli/rrtools-repro-research: Tutorial on Reproducible
    Research in R with rrtools
-   <https://karthik.github.io/holepunch/> \| Configure Your R Project
    for binderhub • hole punch
-   <https://github.com/karthik/holepunch> \| karthik/holepunch: Make
    your R project Binder ready
-   <https://peerj.com/preprints/3192/> \| Packaging data analytical
    work reproducibly using R (and friends) \[PeerJ Preprints\]
-   <https://github.com/alan-turing-institute/the-turing-way/tree/master/workshops/build-a-binderhub>
    \| the-turing-way/workshops/build-a-binderhub at master ·
    alan-turing-institute/the-turing-way
-   <https://github.com/alan-turing-institute/the-turing-way/tree/master/workshops>
    \| the-turing-way/workshops at master ·
    alan-turing-institute/the-turing-way
-   <https://research-compendium.science/> \| Research Compendium
-   <http://inundata.org/talks/rstd19/#/0/33> \|
    reproducible-data-analysis
-   <https://github.com/benmarwick/rrtools> \| benmarwick/rrtools:
    rrtools: Tools for Writing Reproducible Research in R
-   <https://github.com/shrysr/correlationfunnel> \|
    shrysr/correlationfunnel: Speed Up Exploratory Data Analysis (EDA)
-   <https://github.com/cboettig/nonparametric-bayes> \|
    cboettig/nonparametric-bayes: Non-parametric Bayesian Inference for
    Conservation Decisions
-   <https://lincolnmullen.com/blog/makefiles-for-writing-data-analysis-ocr-and-converting-shapefiles/#fnref2>
    \| Makefiles for Writing, Data Analysis, OCR, and Converting
    Shapefiles \| Lincoln Mullen
-   <https://github.com/lmullen/civil-procedure-codes/blob/master/Makefile>
    \| civil-procedure-codes/Makefile at master ·
    lmullen/civil-procedure-codes

[TODO]{.todo .TODO} Dabbling with Linux helps you become better at data science {#dabbling-with-linux-helps-you-become-better-at-data-science}
-------------------------------------------------------------------------------

My first real introduction to Linux was when I had to run CFD
(Computational Fluid Dynamics) simulations on a Linux based computing
cluster during my Master\'s thesis. Upto this point, I was aware of
Linux and Open Source, but had never taken the time to dabble in it.

The only way to access it was via SSH and that was when I was introduced
to the `tail -f` command to monitor the logs of the simulation file.

This was utterly fascinating to me: so much information could be
obtained just from a terminal, using the command line.

Over the years, I\'ve managed to gather a lot more expertise in using
Linux in vaguely related bits and pieces. The journey was certainly not
easy, so much so that I captured as much information as I could on the
CFD-Online Wiki and as you can see, that is focused more on open source
CFD applications.

The above was accelerated significantly when I started my foray into
Emacs, around 4 years ago. Emacs works great on Linux machines, and it
was so easy to install libraries and applications using the command
line.

[TODO]{.todo .TODO} Function to search a column and flag columns {#function-to-search-a-column-and-flag-columns}
----------------------------------------------------------------

This is a function I wrote to extract component dimensions from an
irregularly formatted inventory and sales database.

One thing I would like to improve in the code is that the mutate is
automatically mapped to any number of specified columns, instead of the
manual specification of `str_detect` for each column.

``` {.r org-language="R"}
##' Dimension extraction
##' @param data, first_cat, cat_match, search, value, dimx, col1 , col2
##' @return
##' @description Matches a specified category (cat_match) with an existing category (first_cat). Searches for a term (search) through col1 and col2, and if there is a match, the function will mutate the speified column (dimx) with a =value=.

dim_extract <- function(data,
                        first_cat,
                        cat_match,
                        search,
                        value,
                        dimx = dim_1,
                        col1 = ItemName,
                        col2 = ItemDescription) {
  dimx <- enquo(dimx)
  dimx_name <-  quo_name(dimx)
  first_cat <- enquo(first_cat)
  col1 <- enquo(col1)
  col2 <- enquo(col2)
  data %>%
    mutate(!!dimx_name := case_when(
    ((!!first_cat) == cat_match) & (str_detect((!!col1), search)) ~ value,
    ((!!first_cat) == cat_match) & (str_detect((!!col2), search)) ~ value,
    TRUE ~ !!dimx
    )
  )
}

purchase_order_filtered_tbl  %>%
filter(!duplicated(ItemName))
  dim_extract(ini_category, "reducer", "((?i)(x|to)1/4|(?i)x 1/4)\"", "250", dimx = dim_2) %>%
```

Method: regex search for a pattern in a specified column or columns and
populate another column with say a category that you specify.

[TODO]{.todo .TODO} Setting up Continuous Integration (CI) for docker containers {#setting-up-continuous-integration-ci-for-docker-containers}
--------------------------------------------------------------------------------

This blog post takes you through the process of setting up Continuous
Integration for building docker images via Dockerhub and Github, and via
Github Actions. It also contains a condensed summary of important notes
from the documentation.

Goal: Understand the principles of CI and actually use it to get
automated builds of the docker images that I have built for my
datascience toolbox.

Essentially I want to be able to a status check the docker containers
that I am maintaining. Eventually I want to setup a series of checks
that the libraries and software tools that I use are working as
expected. Though dockerhub enables containers to be built on a commit, I
would also like a CI/CD pipeline to be setup in order to understand how
it actually works.

### Plan \[3/3\]

1.  \[-\] Complete reading the [Github documentation on github
    actions](https://help.github.com/en/actions).
    1.  \[X\] setup a github hosted runner
    2.  \[-\] Setup a self-hosted runner : ~~Lower priority~~ Postponed
        because it is better to understand how a runner works with
        github code before allowing any github code to run on my VPS.
2.  \[X\] Create a CI integration between Dockerhub and Github
3.  \[X\] Expand the CI setup to the datascience docker containers.

### Setting up a Github Runner \[0/1\]

A github runner is essentially creatd by using the Actions tab. There is
a marketplace of Actions that can be used for free. Actions already
exist for many popular workflows like building a docker image and
pushing it to some registry.

Apparently the first action has to be a checkout of the repository.
Without this step, the process will not work. I spent a long time in

Specify build context to a specific folder. i.e do not use `build .`
because then the context and paths will not work, thus the `COPY` type
functions won\'t work.

Apparently Github will reocognise yaml files only within the
sanctimonious `.github/workflows` location, though I may be wrong. If
their autosuggestion is used, the folder is created automatically.
However, thankfully,any YAML file created in this folder will be run by
Github actions. Refer to the notes below regarding the API limitations.
Since it appears that this YAML file cannot be used for say Travis or
CircleCI, it may be good to have these within the github folder anyhow.

Getting started was as simple as using the actions tab when the
repository is opened. A basic YAML template is offered and was actually
sufficient for me to quickly get started.

The build context is specified by the location of the file in this case,
and the tag can be specified. Currently, I\'m using an ephemeral
container.

-   \[ \] An idea for a test would be like : run an ubuntu docker
    container, and then call the shiny container within it. Get the
    container running, and then also devise some output via R scripts.
    One way could be load a bunch of packages. Another way could be to
    get a shiny app to run and provide some kind of temporary output.
    This has to be refined further.

``` {.yaml}
name: Docker Image CI

on: [push]

jobs:

  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: shiny
      run:  docker build . --file /shiny/Dockerfile --tag my-image-name:$(date +%s)q
```

The build details can be found by clicking on the individual jobs. The
raw log can be downloaded for verbose logs to enable a good text search.
During a live build process, there is some lag between the log update
and the webpage refresh, however it seems within tolerable limits as of
now.

![](~/hugo-sr/static/img/github-actions-list-of-builds.png)

![](~/hugo-sr/static/img/detailed-build-info-github-actions.png)

1.  Notes about build and queue

    The free build time is rather long and the configuration of the
    servers is unknown. It is probably a lot faster to build the images
    locally and push them to dockerhub. However, minor image updates and
    small image builds are quite quick to take place. The key is getting
    a single successful build off a github commit after which the
    context is established and new layers need not take the same amount
    of time.

    All this being said, the queue time in dockerhub is very long
    compared to the queue time of builds via actions on Github. The free
    tier is actually quite generous for a lot of room to play and
    experiment. It would also appear that the github computer servers
    are faster than Dockerhub.

### Setting up CI via Dockerhub and Github

The pre-requisite is of course that you have a docker image in your
repository.

This process is relatively simple. Login to your Dockerhub account and
click on the fingerprint like icon to reach the account settings. Use
the linked accounts tab and setup github with your login credentials.

![](~/hugo-sr/static/img/account-settings-dockerhub.png)

Next, select your docker image repository and click on the Builds tab
and click on setup automated builds.

![](~/hugo-sr/static/img/configure-automated-build-dockerhub.png)

Now you have the option to select a github repository and settings are
available to point to a particular branch or a particular Dockerfile as
well.

![](~/hugo-sr/static/img/autobuild-configuration-github.png)

Note the option Enable for Base Image for the Repository Links. This can
be set enabled if your image depends on another image. Suppose that base
image is updated, then a build will be triggered for your image.

The source option can be set to a named branch or a tag and the docker
tag must also be specified. The build context helps if you have
[multiple docker configurations stored
together](https://github.com/shrysr/sr-ds-docker).

Note also below that Environment variables can be specified thus enabled
a more customised deployment of the image. The variable can be used to
specify things like the username and password, Rstudio version or r-base
version, etc. Docker image tags are typically used to demarcate these
more easily.

### General notes

1.  Overview of components

    For any code to run, there has to be a server or a computer to run
    it. This is called a runner and one can be created on a self-hosted
    platform or there are services with different tiers on which the
    runners can be placed.

    > Reference: [Github
    > docs](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/about-github-actions)
    >
    > GitHub Actions enables you to create custom software development
    > life cycle (SDLC) workflows directly in your GitHub repository.
    > GitHub Actions is available with GitHub Free, GitHub Pro, GitHub
    > Team, GitHub Enterprise Cloud, and GitHub One
    >
    > Workflows run in Linux, macOS, Windows, and containers on
    > GitHub-hosted machines, called \'runners\'. Alternatively, you can
    > also host your own runners to run workflows on machines you own or
    > manage. For more information see, \"About self-hosted runners.\"
    >
    > You can create workflows using actions defined in your repository,
    > open source actions in a public repository on GitHub, or a
    > published Docker container image. Workflows in forked repositories
    > don\'t run by default.
    >
    > You can create a workflow file configured to run on specific
    > events. For more information, see \"Configuring a workflow\" and
    > \"Workflow syntax for GitHub Actions\".
    >
    > GitHub Marketplace is a central location for you to find, share,
    > and use actions built by the GitHub community.

2.  On API limits

    Most of these limits are **not applicable** to self-hosted runners.
    Exception to the above : \"You can execute up to 1000 API requests
    in an hour across all actions within a repository.\"

    > There are some limits on GitHub Actions usage. Unless specified,
    > the following limits apply only to GitHub-hosted runners, and not
    > self-hosted runners. These limits are subject to change.
    >
    > -   You can execute up to 20 workflows concurrently per
    >     repository. If exceeded, any additional workflows are queued.
    >
    > -   Each job in a workflow can run for up to 6 hours of execution
    >     time. If a job reaches this limit, the job is terminated and
    >     fails to complete.
    >
    > -   The number of concurrent jobs you can run across all
    >     repositories in your account depends on your GitHub plan. If
    >     exceeded, any additional jobs are queued.
    >
    >       GitHub plan   Total concurrent jobs   Maximum concurrent macOS jobs
    >       ------------- ----------------------- -------------------------------
    >       Free          20                      5
    >       Pro           40                      5
    >       Team          60                      5
    >       Enterprise    180                     15
    >
    > -   You can execute up to 1000 API requests in an hour across all
    >     actions within a repository. This limit also applies to
    >     self-hosted runners. If exceeded, additional API calls will
    >     fail, which might cause jobs to fail.
    >
    >     In addition to these limits, GitHub Actions should not be used
    >     for:
    >
    > -   Content or activity that is illegal or otherwise prohibited by
    >     our Terms of Service or Community Guidelines.
    > -   Cryptomining
    > -   Serverless computing
    > -   Activity that compromises GitHub users or GitHub services.
    > -   Any other activity unrelated to the production, testing,
    >     deployment, or publication of the software project associated
    >     with the repository where GitHub Actions are used. In other
    >     words, be cool, don't use GitHub Actions in ways you know you
    >     shouldn't.
    >
    > You can execute up to 1000 API requests in an hour across all
    > actions within a repository. This limit also applies to
    > self-hosted runners. If exceeded, additional API calls will fail,
    > which might cause jobs to fail.

3.  Self-hosted runners versus github hosted runners

    > Self-hosted runners can be physical, virtual, container,
    > on-premises, or in a cloud. You can use any machine as a
    > self-hosted runner as long at it meets these requirements:
    >
    > -   You can install and run the GitHub Actions runner application
    >     on the machine. For more information, see \"Supported
    >     operating systems for self-hosted runners.\"
    > -   The machine can communicate with GitHub Actions. For more
    >     information, see \"Communication between self-hosted runners
    >     and GitHub.\"
    >
    > GitHub-hosted runners offer a quicker, simpler way to run your
    > workflows, while self-hosted runners are a highly configurable way
    > to run workflows in your own custom environment.
    >
    > -   GitHub-hosted runners:
    >     -   Are automatically updated.
    >     -   Are managed and maintained by GitHub.
    >     -   Provide a clean instance for every job execution.
    > -   Self-hosted runners:
    >     -   Can use cloud services or local machines that you already
    >         pay for.
    >     -   Are customizable to your hardware, operating system,
    >         software, and security requirements.
    >     -   Don\'t need to have a clean instance for every job
    >         execution.
    >     -   Depending on your usage, can be more cost-effective than
    >         GitHub-hosted runners.

    Notes on setting up :Using an .env file

    > If setting environment variables is not practical, you can set the
    > proxy configuration variables in a file named .env in the
    > self-hosted runner application directory. For example, this might
    > be necessary if you want to configure the runner application as a
    > service under a system account. When the runner application
    > starts, it reads the variables set in .env for the proxy
    > configuration.
    >
    > An example .env proxy configuration is shown below:
    >
    > ``` {.text}
    > https_proxy=http://proxy.local:8080
    > no_proxy=example.com,myserver.local:443
    > ```

    **Self-hosted runner security** : With respect to public repos -
    essentially, this means that some repo of code on github is able to
    run on your computer. Therefore unless the code is trusted and
    vetted, it is dangerous to allow this. Forks can cause malicious
    workflows to run by opening a pull request.

    It is safer to use public repositories with a github hosted runner.

    > We recommend that you do not use self-hosted runners with public
    > repositories.
    >
    > Forks of your public repository can potentially run dangerous code
    > on your self-hosted runner machine by creating a pull request that
    > executes the code in a workflow.
    >
    > This is not an issue with GitHub-hosted runners because each
    > GitHub-hosted runner is always a clean isolated virtual machine,
    > and it is destroyed at the end of the job execution.
    >
    > Untrusted workflows running on your self-hosted runner poses
    > significant security risks for your machine and network
    > environment, especially if your machine persists its environment
    > between jobs. Some of the risks include:
    >
    > -   Malicious programs running on the machine.
    > -   Escaping the machine\'s runner sandbox.
    > -   Exposing access to the machine\'s network environment.
    > -   Persisting unwanted or dangerous data on the machine.

4.  CI

    Continuous Integration: enables commits to trigger builds and
    thereby enhances error discovery and rectification process.

    The CI server can run on the same server as the runner. Therefore it
    can be github hosted or a self-hosted CI server.

    Github analyses a repository when CI is setup and recommends basic
    templates depending on the language used.

    > When you commit code to your repository, you can continuously
    > build and test the code to make sure that the commit doesn\'t
    > introduce errors. Your tests can include code linters (which check
    > style formatting), security checks, code coverage, functional
    > tests, and other custom checks.
    >
    > Building and testing your code requires a server. You can build
    > and test updates locally before pushing code to a repository, or
    > you can use a CI server that checks for new code commits in a
    > repository.
    >
    > You can configure your CI workflow to run when a GitHub event
    > occurs (for example, when new code is pushed to your repository),
    > on a set schedule, or when an external event occurs using the
    > repository dispatch webhook.

General
=======

[DONE]{.done .DONE} mosh - for better access to my VPS [[vps]{.smallcaps}]{.tag tag-name="vps"} [[mosh]{.smallcaps}]{.tag tag-name="mosh"} {#mosh---for-better-access-to-my-vps}
------------------------------------------------------------------------------------------------------------------------------------------

[Mosh](https://mosh.org/) is short for mobile shell, and is useful as an
alternative to SSH, especially for poor network conditions, and where
one has to frequently switch networks. It works via the UDP port, which
has to be specifically enabled. I learnt of mosh through the guys in the
\#emacs.

I\'ve faced frequent trouble due to network issues over SSH connections,
with the lag hampering my ability to type, in general, and it is
particularly inconvenient to respond on IRC/Weechat. I\'m hoping mosh
will alleviate the issue.

UDP needs to be enabled for mosh to work. I used UFW on the ports
60000:61000 for this.

``` {.shell}
sudo ufw allow 60000:61000/udp
```

-   Essentially, a mosh server runs on both the machines (VPS and local
    machine), and these perform the background job of syncing commands
    and output with each other. This reduces the lag in typing, among
    other advantages. The initial connection of Mosh, including
    authentication is via SSH, after which the UDP protocol is used.

Installing Mosh:

On debian - mosh is directly available as a package. Run
`apt-get update` and then install mosh.

``` {.shell}
apt-get install "mosh"
```

The `mosh-server` has to be run on both the machines. It may be a good
idea to include this in `.bashrc`, or in the list of start-up programs.
This command will start up the mosh-server and detach the process (into
the background).

``` {.shell}
mosh-server
```

This is where I ran into trouble. A UTF-8 environment has to be
specified for mosh to run, and it appears that the locales of the two
connecting machines have to match (?). On Debian, this is relatively
easy with `dpkg`

``` {.shell}
sudo dpkg-reconfigure locales
```

I chose the `en_USA.UTF-8` option. The existing locale configuration can
be viewed with `locale`.

``` {.shell}
locale
```

Sometimes, additional settings for the locale are defined in locations
like `~/.bashrc`. This should be something like :

``` {.shell}
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
# export LC_ALL=en_US.UTF-8
```

The above can be used for explicitly setting the preference. The [Debian
wiki](https://wiki.debian.org/Locale) dissuades end-users from using
`LC_ALL`, but that is easiest way. My initial settings were with
`en_CA.UTF-8`. While this is also UTF-8, for some reason, mosh still
threw out locale errors. In any case, I wanted all my computers to
uniformly use the `en_US` version.

### Did mosh make a difference?

It\'s only been a few hours, but the difference can already be felt.
Mosh clearly indicates when the connection has been lost and there is no
lag in typing. Further experimentation is necessary to understand its
behavior, but atleast, I can type out a message in peace without lag.

\[2019-07-31 Wed\] After 3+ days of using mosh, I am happy to note that
the experience of engaging with my vps over a terminal has significantly
improved. There were few instances of really poor network connection,
and mosh would clearly indicate the disconnection, and also allow a safe
exit if required. I can switch computers and jump right in, without
bothering to restore the SSH connection.

[DONE]{.done .DONE} Implementing HTTPS : Let\'s Encrypt {#implementing-https-lets-encrypt}
-------------------------------------------------------

### What is Let\'s Encrypt?

Let\'s Encrypt is a Certificate Authority (CA). A certificate from a CA
is required to enable HTTPS.

Certbot\'s documentation summarises it well:

> Certbot is part of EFF's effort to encrypt the entire Internet. Secure
> communication over the Web relies on HTTPS, which requires the use of
> a digital certificate that lets browsers verify the identity of web
> servers (e.g., is that really google.com?). Web servers obtain their
> certificates from trusted third parties called certificate authorities
> (CAs).

### How Let\'s Encrypt works

-   To certify my domain, I need to demonstrate control over my domain.
    i.e one has to run a software tool to generate this certificate
    (periodically) on the server. being able to do this demonstratesa
    control over the domain.
    -   Similar to domain control, there are other certificates for
        different purposes as well. See the excerpt from the ACME
        protocol below:

> Different types of certificates reflect different kinds of CA
> verification of information about the certificate subject. \"Domain
> Validation\" (DV) certificates are by far the most common type. For DV
> validation, the CA merely verifies that the requester has effective
> control of the web server and/or DNS server for the domain, but does
> not explicitly attempt to verify their real-world identity. (This is
> as opposed to \"Organization Validation\" (OV) and \"Extended
> Validation\" (EV) certificates, where the process is intended to also
> verify the real-world identity of the requester.)

-   Let\'s Encrypt\'s
    [documentation](https://letsencrypt.org/getting-started/) mentions
    that the software above will use the
    [ACME](https://ietf-wg-acme.github.io/acme/draft-ietf-acme-acme.txt)
    protocols to generate the cert, and there are different approaches
    to do so, depending on the availability of shell access (or not) to
    the server.
-   ACME stands for Automatic Certificate Management Environment : The
    [introduction](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-1)
    in the RFC demonstrates how ACME automates a significantly manual
    procedure combining ad-hoc protocols.

> ...protocol that a certificate authority (CA) and an applicant can use
> to automate the process of verification and certificate issuance. The
> protocol also provides facilities for other certificate management
> functions, such as certificate revocation.

-   Since I have shell access to my VPS, I will focus on this approach.
-   There are [multiple ACME
    clients](https://letsencrypt.org/docs/client-options/) to choose
    from, and [Certbot](https://certbot.eff.org/) is \'recommended\' (by
    the EFF). On a superficial glance,
    [GetSSL](https://github.com/srvrco/getssl/tree/APIv2) looks
    interesting as an alternative.

> At this point, I will proceed with Certbot, because I\'ve not yet
> found any particular reason not to.

### On Certbot \[0/1\]

The [Certbot website](https://certbot.eff.org/all-instructions) provides
customized instructions for the OS and server. The main requirement(s)
is having an online HTTP website with an open port 80, hosted on a
server. I can go ahead since I\'ve got these.

> Certbot will run on the web server (not locally) periodically and will
> help in automating the process of certificate management.

Setting up Certbot (on debian)

``` {.shell}
wget https://dl.eff.org/certbot-auto
sudo mv certbot-auto /usr/local/bin/certbot-auto
sudo chown root /usr/local/bin/certbot-auto
sudo chmod 0755 /usr/local/bin/certbot-auto
```

Checking that the above was actually done with a simple:

``` {.shell}
ls -al /usr/local/bin/cert*
```

Next, a one-command certificate setup is possible (with nginx)

> Note that this command may require additional dependencies to be
> installed, and will need a bunch of user input as well, and so should
> not be run in a dumb terminal.

``` {.shell}
sudo /usr/local/bin/certbot-auto --nginx
```

This will:

-   Install necessary dependencies and the certbot plugins
    (authenticator, installer) for nginx.

> Noted the option of `--no-boostrap` for debian. I\'m not sure, but
> this probably has to do with addressing the dependencies for different
> debian versions.

For reference, the following packages were checked/installed:

``` {.example}
ca-certificates is already the newest version (20190110).
ca-certificates set to manually installed.
gcc is already the newest version (4:8.3.0-1).
libffi-dev is already the newest version (3.2.1-9).
libffi-dev set to manually installed.
libssl-dev is already the newest version (1.1.1c-1).
openssl is already the newest version (1.1.1c-1).
openssl set to manually installed.
python is already the newest version (2.7.16-1).
python-dev is already the newest version (2.7.16-1).
python-virtualenv is already the newest version (15.1.0+ds-2).
virtualenv is already the newest version (15.1.0+ds-2).
virtualenv set to manually installed.

Suggested packages:

augeas-doc augeas-tools
The following NEW packages will be installed:
  augeas-lenses libaugeas0
```

An email address has to be entered for \'urgent\' communication
regarding the certificate, and optionally can be shared with the EFF
(which was a trifle annoying (as a part of an installation process),
though I said yes).

> I had to enable https with UFW to complete the test successfully.
> `sudo ufw allow https`. Earlier, only HTTP had been enabled.

Automatic certificate renewal by setting up a cron job.

``` {.shell}
echo "0 0,12 * * * root python -c 'import random; import time; time.sleep(random.random() * 3600)' && /usr/local/bin/certbot-auto renew" | sudo tee -a /etc/crontab > /dev/null
```

-   \[ \] deciphering the cron job, and verifying it is as expected. For
    now, I\'ve not run this command because I want to know what it is
    doing first.

As an alternative to a \'one-step\' installation, getting just the
certificate will mean nginx\'s configuration will have to done manually.
This is probably a good choice to \'learn more\'.

``` {.shell}
sudo /usr/local/bin/certbot-auto certonly --nginx
```

> I need to verify this, but it appears nginx\'s main configuration is
> at `/etc/nginx/nginx.conf` , and a quick peek showed me that the user
> was still set as \'www-data\', which was used as the initial setup of
> the nginx test website. This was changed subsequently. Perhaps this is
> why I am unable to get Wordpress plugins write access.

At the end of it all, I received a
[link](https://www.ssllabs.com/ssltest/analyze.html?d=s.ragavan.co),
through which it appears I can get a detailed \'SSL Report\'.

``` {.example}
Congratulations! You have successfully enabled https://s.ragavan.co

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=s.ragavan.co
```

> This report appears to be quite important, but I could not make much
> sense of it, and it needs to be re-visited. As such, I see that the
> [ACME](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-7)
> challenges need to be understood to comprehend these results.

### Short Peek under the hood.

A skim of the extensive [documentation of
Certbot](https://certbot.eff.org/docs/) shows that certbot relies on [2
types of plugins](https://certbot.eff.org/docs/using.html#plugins) to
function.

1.  authenticators: plugins to obtain a certificate, but not install
    (i.e edit the server configuration). Used with the `certonly`
    command.
2.  Installers: used to modify the server\'s configuration. Used with
    the `install` command.
3.  Authenticators + installers : can be used with the `certbot run`
    command.

These plugins use \'[ACME Protocol
challenges](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-7)\'
to prove domain ownership. Section 7 (as of today) of the internet draft
of the standard provides an overview, and the challenges are described
in detail in the draft.

> There are few types of identifiers in the world for which there is a
> standardized mechanism to prove possession of a given identifier. In
> all practical cases, CAs rely on a variety of means to test whether an
> entity applying for a certificate with a given identifier actually
> controls that identifier.
>
> Challenges provide the server with assurance that an account key
> holder is also the entity that controls an identifier. For each type
> of challenge, it must be the case that in order for an entity to
> successfully complete the challenge the entity must both:
>
> -   Hold the private key of the account key pair used to respond to
>     the challenge
>
> -   Control the identifier in question
>
### Conclusions

-   HTTPS via Let\'s Encrypt is setup for my website. Come visit at
    <https://s.ragavan.co>
-   Had a brief introduction into the methodology/philosophy behind
    Let\'s Encrypt.
-   Brief exploration of ACME and it was quite interesting to go through
    the draft standard, though it will take a lot more effort to fully
    comprehend all the tests. I think it is likely that I have visit
    this in more detail as I make progress in learning about encryption.
-   Learned about the existence of \'[Internet
    Standards](https://en.wikipedia.org/wiki/Internet_Standard)\'. These
    are documented by one or more documents called RFC\'s (Request for
    Comments) and revised until deemed satisfactory to become a
    standard.

[DONE]{.done .DONE} Incremental improvements can lead to significant gains [[Productivity]{.smallcaps}]{.tag tag-name="Productivity"} [[Emacs]{.smallcaps}]{.tag tag-name="Emacs"} [[Orgmode]{.smallcaps}]{.tag tag-name="Orgmode"} {#incremental-improvements-can-lead-to-significant-gains}
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

While reading the book [Atomic Habits by James
Clear](https://jamesclear.com/atomic-habits), I was reflecting that my
choice of embracing [Emacs](https://www.gnu.org/software/emacs/) and
progressively gaining mastery over it was intimately connected with the
philosophy preached in the book.

My efforts initially started out with a craving for a system to quantify
and manage my tasks, habits, notes, blog writing, job applications and
projects in a custom environment, and to be able to build toolkits of
code to perform repetitive tasks. As mentioned in an [earlier blog
post](https://s.ragavan.co/2019/08/getting-productive-an-exploration-into-holistic-task-management/),
I tried several approaches before settling on Emacs. The idea was to
find or create a single system to track everything of importance in my
life (with ease and efficiency). This was instead of a fragmented
approach of using multiple tools and techniques, for example, Sublime
Text / Atom as a text editor and [Todoist](https://todoist.com/?lang=en)
as a task management tool.

I started with a vanilla configuration of Emacs and painstakingly
borrowed (and eventually) modified lisp snippets to implement desired
\'features\' or behaviors. It was a just a couple of features every
week, initially focused on Org mode\'s behavior alone. That was nearly 3
years ago. As of now, I am able to manage my blog \[hugo\], view my
email \[mu4e\], browse the web \[w3m\], seamlessly capture notes / ideas
/ tasks from (almost) anywhere \[Org mode\], chat on IRC, build
multi-language code notebooks with ease \[Org babel\]. All the above
provide me significant advantages in speed and efficiency which still
have plenty of potential to improve.

Sure, I certainly appear closer to my goal today.. however, I did not
know if it was a pipe dream when I started out. It was often extremely
frustrating, right from memorizing the \'crazy\' keybindings in Emacs,
to struggling with getting a lisp snippet to work as expected.

Choosing Emacs had unexpected rewards as well. For example, the need of
synchronizing my notes and Emacs configuration with multiple machines
led me to Git. [Magit\'s](https://magit.vc/) easily accessible commands
and relatively visual interface has been a massive help in getting
things done with Git, despite not having any deep technical knowledge of
how Git works.

My journey with Emacs is testament that an incremental, compounding
improvement over time can ultimately result in significant gains. It is
also important to acknowledge that I am standing on the shoulder of
giants and the awesome [Scimax](https://github.com/jkitchin/scimax) is a
cornerstone in my toolkit.

[DONE]{.done .DONE} Getting productive - an exploration into holistic task management [[Productivity]{.smallcaps}]{.tag tag-name="Productivity"} [[Orgmode]{.smallcaps}]{.tag tag-name="Orgmode"} [[Emacs]{.smallcaps}]{.tag tag-name="Emacs"} {#getting-productive---an-exploration-into-holistic-task-management}
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Introduction

To integrate tasks, reminders, notes, coding workflow into a single
framework is no easy challenge. Org mode and Emacs help you do just
that.

After trying out several tools, IMHO : [Todoist](https://todoist.com)
offers the best bang for your buck, especially with it\'s natural
language parsing ability, smooth and reliable sync as well as its
multi-platform availability. Many describe
[Omnifocus](https://www.omnigroup.com/omnifocus) to be the king of task
management tools, with dedicated apps for different purposes and
probably well integrated.

My journey veered away from Omnifocus since it is limited to the Apple
platform and this is obviously a serious handicap for people (like me)
who are often forced to use multiple operating systems and devices
distributed between personal and work environments.

I\'d religiously managed my tasks on Todoist for over a year via the
Chrome extensions/add-ins, the stand alone apps on Windows and the Mac,
and on Android as well as iOS.

However, there was something missing in terms of being able to truly
capture it all. This led me to [Emacs](https://www.gnu.org/s/emacs/). My
search is summarised in this article.

### Needs versus the software development

The real problem surfaced when my needs evolved at a pace and
specificity that a general software\'s development could not cater to.
The problem is characterized by an endless wait for seemingly simple
features that could make a phenomenal difference to personal workflow
and productivity. This feature may range from a small tweak or bugfix to
a rewiring of the basic behavior of the program itself.

Additionally, the proprietary format of tasks/notes and entries in
Todoist or even Evernote is not a comforting aspect. On the other hand,
using a simple text file with lists of work or notes is too simplistic
to address a complex problem.

However, the issue could be resolved when the simple and ubiquitous Text
file is parsed by a system like Org mode with in built and novel
routines to filter and present the data in the text file in a very
useful. Ultimately the key factor is that the workflow and output can be
completely customised as required.

### Things I\'d like from a task management tool:

1.  Rapid and seamless Task/Note taking ability - could be generic, or
    specific to a particular project/task.
2.  Quick capturing of links and snippets from websites and emails
3.  Consistent experience across multiple platforms and very fast sync.
4.  Ability to manage personal or work related projects
5.  A date management system with atleast reasonably good understanding
    of natural language
6.  Refiling tasks/notes very easily across main tasks or categories or
    projects
7.  Customisable Views of the task summary along with the deadlines
8.  Task and Note search and filtering at every level possible
9.  Ability to easily export notes to multiple formats and write in some
    form of markup language so as to take care of formatting on the go.
10. Preferably an all-in-one tool for managing notes, all kinds of
    writing, research, tasks, recurring reminders, maintaining an
    activity log/journal, project summaries .. etc.
11. Includes \'clocking\' abilities for tasks.
12. Fast keyboard based shortcuts and \'bookmarks\' to do all that is
    required.
13. Recording tasks or notes from the phone, while on the go.
14. Should have the lightest footprint possible in terms of time spent
    on the tool, as well as system resources with no compromise in
    benefits derived.

### Can it be achieved?

Short answer: Yes. Through Emacs.

Sure, several of the above points can be done in Todoist and other
tools, in one way or via combining different services.

However, a holistic consideration of the above points indicate a system
that is a cross between Todoist and Evernote, capable of being utilised
for a multitude of purposes : a customised GTD workflow plus an
organiser for notes or writings. Point no 9, could serve to be a concise
but incomplete statement of Orgmode\'s capabilities, and is a stark
reminder of Todoist\'s specific expertise in only task management.
Additionally, the above points can be done in orgmode, *very*, *very*
quickly. Evernote has a great system, but is not as fast, because it
indexes a huge variety of content. [^18]

### Examples of workflows

Lets say that while typing up a project summary, I remember an
additional task for another project or perhaps need to note down a
snippet of generic information. To compensate for the lack of a
photographic memory without breaking my on-going workflow - I need to be
able to store the task/note/idea in a place that I can easily look up
for further processing.

Such an activity is not at all streamlined with Todoist, and definitely
not so with Evernote. With Org mode its just a `C-c c`, or Control + c
and hit c again. Optionally, a `C-cw` for refiling the note on the spot
if desired. When I hit refile - I can search through my org headings or
projects and place the newly captured item exactly where it should be.

Once accustomed to the speed of recording stuff with Org-capture, along
with the myriad possibilities of auto-save, backups, moving the cursor
to the last location you were at, switching to another document/heading
at lightning speed and etc - it will be hard to find another system that
is truly competitive.

Project management via Emacs using the excellent
[projectile](https://github.com/bbatsov/projectile) package can enable
you to find information at a speed that is very pleasing. I have often
needed to deal with several customers of different kinds, thoroughly
understand their requirements, resolve technical and commercial
ambiguities and be able to refer to earlier jobs where something was
agreed upon. I\'ve often worked in projects with a bewildering number of
aspects to take care of, along with sporadic infusions of information
which could be clarifications or even new information altogether.

Included in project / productivity /relationship management are several
subsets of activities like Minutes of Meetings (MOM\'s), summaries of
travel/visits to the customer, telephonic discussions, indications of
future projects as well as generic or specific problems.

Using Org mode, it is possible create customised workflows and templates
to manage all the above aspects, more than any other note taking system,
including only handwritten notes. An excellent, comprehensive overview
can be found in [Bert Hansen\'s
article](http://doc.norang.ca/org-mode.html).

### Everybody\'s needs are unique

Eventually, I guess we all come to realise the fact that each human
being is truly unique. Each one of us have our own ways of thinking,
being and approaching problems.

While Todoist worked very well for me - I was still bothered by being
constrained by it\'s proprietary format and the lack of a lifetime
membership with a one time payment. Money spent should give me a tool
that brings supreme value and satisfaction with it. It was also tiresome
to take detailed notes on tasks and rely on a separate
Simplenote/Evernote system via Sublime Text for this purpose. You may
have a different viewpoint. You may want a great GUI design and app that
works well on your phone in addition to other environments. [^19]

Orgmode is more aligned to people who prefer to get most of their work
done on their computers, who are or atleast don\'t mind being keyboard
shortcut freaks and those who would like to take the effort to learn a
souped up text editor like Emacs that can evolve to cover a lot of needs
efficiently. It\'s not going to work well for people who need a reminder
to pop up on their phones, with a fancy GUI and those who expect a
software to work extremely well right out of the box. However, this *is*
Org mode and Emacs.... there are ways to sync your iOS / outlook
calendar with orgmode\'s calendar, or with wunderlist or Toodledo.
Anything is possible, but it just won\'t be via some classy GUI..

### Concluding points

While it may seem daunting at first - the feeling of being able to
search through existing notes to know whether you have met this
particular thought/aspect before, can be extremely valuable and very
satisfying. There are people like [Sacha
Chua](http://sachachua.com/blog/) and [Bert
Hansen](http://doc.norang.ca/org-mode.html), who\'ve built complex,
efficient, and beautiful workflows through which a great deal of
achievement has been made possible using the resulting streamlined tool.
As [Cal Newport](http://calnewport.com/) often reiterates in his blog
and exploration on productivity - it is important to be able to
accurately quantify the time being spent on different things. The
[awesome-emacs](https://github.com/emacs-tw/awesome-emacs) list on
github offers several worthy resources, along with the excellent [Planet
Emacsen](http://planet.emacsen.org/).

The organiser tool by itself should have the lightest possible footprint
in terms of the time taken to enter in stuff. Certainly - most people
spend a lifetime in customising emacs and that may seem contrary to the
previous point. However, it is possible to quickly reach a certain point
that results in a marked improvement in productivity and workflow.
Beyond this, leisure time can always be spent in fine-tuning the basic
setup and understanding the code better.

The customisation options with Emacs and Org mode are literally endless
and constrained only by programming skills, or Googling skills to find
the code snippet that can get your work done, not to mention social
skills in getting help via online communities. This is actually a lot
easier than it sounds. While a bunch of people would call this a
weakness, there are a large number of people who see the value in a
customised tool which will evolve to facilitate a very fast and
efficient workflow.

Deliberate practise towards improvement is certainly boosted when one is
able to work consistently in a environment customised to needs and
workflows. Using Org mode and Emacs is a firm step in this direction.

[DONE]{.done .DONE} Switching from Evernote to DEVONtechnologies products {#switching-from-evernote-to-devontechnologies-products}
-------------------------------------------------------------------------

I\'ve used [Evernote](https://evernote.com/) since 2014, with over 3k
notes of all kinds stored in it. Though I did try to capture everything
of interest - the procedure was never fast or streamlined enough for me.
The Evernote app runs ridiculously slower on older phones. In
particular, being used to the speed of Emacs and Org mode - I was mostly
displeased with the Evernote Mac / Windows apps as well. I ended up
using the drafts app for writing on iOS devices.

However, using Evernote was still worth due to the availability of an
excellent catch-all bucket for multiple kinds of information, that can
be searched on demand. I could literally whip up important receipts or
scanned copies of a document and it felt wonderful to have that kind of
control over your information. This foray was also fueled by the
deficiencies of Emacs in mobile apps and the ability to store and refer
to rich content and several file types.

### Switching to DEVONthink Pro (DTP)

I\'ve recently converted to [DEVONthink
Pro](https://www.devontechnologies.com/products/devonthink/overview.html)
(DTP). Though DTP is Mac / iOS only, I would personally prefer DTP over
Evernote. Some advantages of DTP:

-   blazing fast application response + search on both iOS and Mac.
-   leverages AI to provide interesting connections between notes and
    ideas. Users have leveraged these connections to help generate new
    ideas from unforeseen connections. There\'s more information
    [here](https://www.devontechnologies.com/technology.html).
    -   so far, my experience is that the notes have to be in a
        particular format,I.e one article or principal idea per note to
        enable a sensible matching with other relevant articles. There
        are several incorrect connections also made.
-   Better control over content organisation.
    -   Project and folder creation, including separate databases for
        different kinds of work.
-   One time payment for a major version of the software, along with
    discounted upgrades.
-   Ability to index local folders.
-   using multiple \'databases\' customised to any workflow, along with
    the provision of password protection and syncing to multiple
    sources.
-   ability to confidently store private information based on the
    encryption and custom syncing options available.
-   Ability to store web archives of Linked in posts (or any content).
    This was not always possible with Evernote. The iOS share option of
    clipping to the DEVONthink to go app as a web archive works rather
    well most of the time.
-   The Evernote plug-in for Chrome/ Firefox works relatively slower.
-   connection with DEVONAgent Pro (a fascinating tool dedicated to
    customised and deep web search. More on this on another blog post)
-   Deploy scripts on databases / notes and thus allowing custom
    workflows with particular note categories.
-   DTP can import all your Evernote notes and tags as they are. This
    worked for me in a single attempt.

It\'s actually hard to quantify the benefits of using DTP. There are a
myriad of features within, including the ability to index locations and
script automated workflows.

For most of the part, I found the speed and response of Evernote to be
frustrating. It hindered a streamlined workflow. There are also
additional irritations with respect to the .enex format and being able
to encrypt information.

No doubt, the ubiquity of Evernote in almost all the platforms (except
Linux[^20]) works in its favor. However, the search response with DTP is
incredibly rapid and the note viewing experience of DTP is extremely
smooth. This is on an ancient mid 2010 macbook pro!

It\'s also worth noting that unlike Evernote - I was actually intrigued
enough to correspond with the technical support team of DTP to
understand features like indexing a folder, and their responses have
been prompt and helpful.

The best place to find up to date information is on the
[DEVONtechnologies forum](https://forum.devontechnologies.com/). Even a
deep search on the internet does not lead to many articles about the
DEVONthink technologies products.

### Some caveats of DTP

-   DTP does offer all the flexibility above. However the quality of the
    Evernote webclipper\'s output is better in several cases. The
    uncluttered text grab is not automated well enough. I\'m yet to
    discover the best pattern.
-   Several apps offer Evernote integration as a premium feature.
-   Evernote offers a more \'polished\' and simpler interface and is
    mainstream and available on multiple platforms. The note taking
    editors and capture mechanism is more user friendly.

### DEVONagent Pro (DAP)

DAP is an intriguing bit of software that facilitates deep searches of
the web and developing automated workflows including report development.
Their algorithm filters searches from any number of databases / engines
/ websites to provide the best matches.

One could use this to monitor the website of a competitor for news
announcements. Or crawl Hackernews for the keyword Datascience. It
appears to be a tool that can provide exactly the information that we
seek by processing the information out there in the web.

This includes generation of mind-map esque graphs connecting keywords in
all the search results. I\'m yet to explore more, but it is very
interesting so far, especially to gain an overview of the subject.

### Some Conclusions

Exploring DTP in conjunction with DEVONagent Pro is absolutely a
worthwhile exercise for those relying a lot on information from the
internet for their jobs and work, and those working in an apple
eco-system. It has a steep(er) learning curve, but will transform your
information management. DAP is also a worthy option to explore to deep
search the web on focused topics.

Yes, it is mac only software. I have not been able to find any
equivalent apps on windows. Another reason to stick to the Apple-verse.

The system is addictive and once a good workflow has been built up, it
would be difficult to use anything else.

### Archiving interesting Linked in posts:

One of the most killer features of using the DEVON 2 GO app is the
ability to capture Linked in posts as web archives. Though not optimal,
in terms of the format - it is still extremely useful to rapidly build
up a reference database of web resources.

[DONE]{.done .DONE} Back to RSS [[\@Productivity]{.smallcaps}]{.tag tag-name="@Productivity"} [[Emacs]{.smallcaps}]{.tag tag-name="Emacs"} {#back-to-rss}
------------------------------------------------------------------------------------------------------------------------------------------

### Why use RSS?

Off late, I had been relying more on email based content consumption.
The phenomenally fast search and filtering capabilities that can be
leveraged with [mu4e](https://www.djcbsoftware.nl/code/mu/mu4e.html)
make this easy.

Even with all these filters, it is quite difficult to keep on top of
news and information from different sources. It is actually inconvenient
to mix important emails and correspondence with newsletters and the
like, which arrive by the dozen(s) everyday.

There\'s also a nagging feeling that relevant and \'up to date\'
information is better searched through Google, with a fresh search each
time. This approach invites distractions. One remedy is to link a google
news feed of a search term into your RSS.

I\'ve always liked [RSS](https://en.m.wikipedia.org/wiki/RSS), however,
the exploration made me actually realise that a dedicated RSS reader
could inspire focused reading and aid in retention of information, and
could be a better option than flooding my inbox.

An all-in-one solution for reading RSS feeds with a capable in-built
browser to view images/webpages/videos would be excellent, along with
the ability to sync with multiple services and facilitate capturing
notes.

### Exploration:

Within Emacs - [Elfeed](https://github.com/skeeto/elfeed) (along with
[Elfeed-goodies](https://github.com/algernon/elfeed-goodies)) is a good
option to read feeds and is strongly integrated with Emacs and org-mode.
A single keypress can be programmed to store a link as an org-heading
with a note. This would really be my first choice as I\'ve found it to
work rather well. I can use an org file to easily organise my feeds !

Unfortunately, there seems no easy way to sync completed feeds to my iOS
devices, though [solutions exist for
Android](https://github.com/areina/elfeed-cljsrn). I do spend a lot of
time on my computer, however, it seems I can just read better and faster
on my iPad and therefore a sync to mobile devices is still important.

Though it does not seem to be a mainstream recommendation on reviews
like [the sweet
setup](https://thesweetsetup.com/apps/best-rss-reader-os-x/) :
[Vienna](http://www.vienna-rss.com/) is a reliable solution (open
source!) to consider using to browse RSS feeds on the Mac OS. This comes
with a caveat - some tinkering is required to get it to sync with a
service.Vienna has inbuilt share options to share via Buffer or Twitter.
Side note: I would recommend using [Buffer](https://buffer.com/) to
manage posts on multiple social media sites in a seamless manner.
Buffer\'s free tier should be sufficient for moderate, personal
purposes. I use it to post on Twitter and Linked in simultaneously.

1.  Harvesting information

    The next consideration was harvesting notable information of
    interest from the RSS feeds. If not Emacs, the information has to go
    to [DEVONThink
    Pro](https://www.devontechnologies.com/products/devonthink/overview.html)
    (DTP), which has a handy pull out drawer into which content can be
    dragged. I was able to just drag and drop the article or text
    selection into the DTP drawer. This appears as a URL / bookmark in
    DTP, and can be converted to a formatted note or web archive
    subsequently. A script could probably accomplish this automatically.
    That\'s for a future project.[^21]

    {{\< figure src=\"/img/vienna-dtp-drawer.png\" title=\"Screenshot -
    Vienna + DTP drawer\" \>}}

    Granted, an application external to Emacs (especially without a
    customisable keyboard driven flow) is not the desirable way to do
    things. Most websites usually have an RSS feed or email subscription
    possibility.

2.  Opting for Feedly as a susbcription service and RSS app

    Unfortunately, Vienna had to be abandoned as it felt more sensible
    to opt for a [Feedly](https://feedly.com/) subscription to enable a
    seamless mobile experience. The Feedly app turned out to run
    surprisingly well on my ancient iPad and I can still drag and drop
    entire articles into DTP which come out to be formatted RTFD files
    which could be read and highlighted in leisure. While it may be nice
    to opt for a standalone app in the Mac for RSS feeds, the Feedly app
    satisfies my needs and is also available cross-platform. Note: I use
    the excellent [Unread
    app](https://www.goldenhillsoftware.com/unread/) to read RSS on my
    newer iPhone.

    Besides the numerous sync options, [Feedly](https://feedly.com/)
    provides other interesting features in their pro subscription, like
    setting up Google keyword searching and organising multiple feeds
    into \'boards\'. This will certainly help in enabling some level of
    filtering. The method of organising sources and OPML imports in the
    mac app is a little clunky and not comfortably intuitive, but it is
    usable.

    There\'s [no easy way to use Elfeed as a feedly
    client](https://emacs.stackexchange.com/questions/4138/how-do-i-use-emacs-as-a-feedly-com-client)
    either.

3.  How to cover them all?

    With numerous sources available on most topics - a technique to read
    is of even more importance. Besides leveraging custom boards, it
    seems the best way to consume content is to rapidly sweep through
    the titles and the short descriptions, and in parallel skim through
    articles of interest. If the article (even slightly) feels worth
    recording and reading in detail, I select the entire article and
    drag it into DTP via the drawer for a future session.

    I try to deploy DTP as my primary reading app, because of the
    ability to highlight lines (which are generally available across
    devices). Besides aiding in skimming the article in the future, it
    helps me know I\'ve actually read the article. This is in addition
    to the core ability to use DTP\'s AI algorithms in searching through
    my notes and forming connections between ideas. I also use smart
    groups that show me the articles captured in the last 1 week, 2
    weeks, 3 weeks, which helps me re-visit them in a structured method.
    The latter works rather well as a memory aid.

    {{\< figure src=\"/img/feedly-dtp-screenshot.png\" title=\"Article
    captured from Feedly into DTP\" \>}}

### Future plans?

It would be ideal to setup my own server which will process the RSS
feeds. Perhaps a Raspberry Pi or something else could be employed. This
would be a cost efficient approach for the long term. Such a setup would
enable using Elfeed to source articles from the server and thus sync
with my mobile devices.

For now, I guess I will have to rely on Feedly.

[DONE]{.done .DONE} An SSD can breathe life into old computers [[Productivity]{.smallcaps}]{.tag tag-name="Productivity"} [[Emacs]{.smallcaps}]{.tag tag-name="Emacs"} [[Linux]{.smallcaps}]{.tag tag-name="Linux"} {#an-ssd-can-breathe-life-into-old-computers}
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

It\'s a well known trick that installing a
[SSD](https://www.storagereview.com/ssd_vs_hdd) in place of the
conventional Hard disk can breathe new life into very old machines. My
mid 2010 Macbook Pro is one such example, being over 8 years old.

In particular, within Emacs - `mu4e` responds much more quickly and
there is significantly less lag in searching / accessing emails and
`HTML` rendering.

The other advantage of using a Mac over Linux is that installation and
setup instructions are more often available out the box for the Mac OS
(though this is changing). I have access to dedicated apps including
Evernote, Dash, Spotify, Whatsap, Slack etc on my Mac. This is in
addition to several other high quality apps on the App store.

I do love using Arch Linux and Antergos and the packing management and
rolling OS upgrades are totally cool. However, a little bit of elegance
in the user interface and hardware (being available out of the box) does
ease up the mind and progress. It takes quite a bit of effort to achieve
that unless you are at the level of purely using [Emacs as window
manager](http://www.howardism.org/Technical/Emacs/new-window-manager.html).

On the Mac, it is easy to move around virtual desktops and use the magic
track pad to rapidly switch between applications as well. I\'m sure many
of these \'gimmicks\' may be setup with diligence and due time on Linux
through solutions with varying levels of quality.

However, as of today : it\'s likely I would have struggled with some
aspects on Linux that are readily available on other systems. Evernote
is an example. After hours of searching for an alternate (and
acceptable) solution for software packages that are not yet ported to
Linux, I would quite possibly end up making a compromise. Typically, the
compromises would mean using Electron or Web based versions of apps,
which are often not as powerful as the desktop app, not to mention
inconvenient. A prime example would be Evernote, on Arch Linux. Some
other examples are apps like Word, Pages, Outlook and Excel and so on,
which are more critical.

Ultimately, my preference would be to use a Mac as my daily driver and
play around with Linux on a back up computer. In any case, multiple
Linux distros can be run on [Virtual Box](https://www.virtualbox.org/)
within the Mac.

[DONE]{.done .DONE} Notes from the movie Whiplash [[Movie\_notes]{.smallcaps}]{.tag tag-name="Movie_notes"} [[excellence]{.smallcaps}]{.tag tag-name="excellence"} {#notes-from-the-movie-whiplash}
------------------------------------------------------------------------------------------------------------------------------------------------------------------

[Whiplash:
Wikipedia](https://en.m.wikipedia.org/wiki/Whiplash_%282014_film%29)

Whiplash is a fascinating movie on many levels regarding a topic that
interests me deeply... How to progressively perform, and strive to
become the very best in a chosen field. Personally, I found each step of
the movie riveting and would recommend it to anybody who would find the
above question even mildly interesting. The movie\'s climax was
immensely interesting, inspiring and supported by great acting. At any
rate, the movie induced a blog post !

The story revolves around the mind and life of a student who wants to be
among the greats in his field, and the way he deals with an abrasive,
abusive and unorthodox teacher whose intentions are to bring out the
best in a student. No movie is perfect - while some points in Whiplash
do appear extreme and therefore relatively unrealistic - the overriding
message and theme conveyed certainly rings out clearly, in an engaging
plot.

I could relate to the following pointers from the movie:

### Leverage stress to achieve new levels of insight and performance

The belief of the teacher, that the best performance or attributes
hidden inside a person can come out only via repeated, unexpected and
stressful prodding. I\'m not sure if this works as shown in the movie,
but I have found unexpected insights at times of extreme stress, that
have were taken forward to habits that changed my life. I\'ve even used
extreme stress to \'plunge\' into habits like regular physical workouts.

> On the other hand, the best teachers I\'ve had were extremely
> intelligent and kind, and had a \'knack\' of getting things across
> without stressful prodding. I mean to convey that stress - is not
> entirely a harmful thing, and can be manipulated to achieve things
> that may not be normally possible.

### Weathering criticism

The mental conditioning required to weather and beat intense, sharp,
depressing criticism along with verbal and physical abuse from a mentor
or teacher and use the same as a motive force for self-improvement and
eventually superlative performance. Though there are known examples of
extreme abrasiveness in leaders like Steve Jobs - such an approach would
not be tolerated by most people today.

-   I know other stories of people working under such mentors, striving
    to learn and gain their approval and eventually winning the same.
    These efforts paid off by resulting in skills, thinking patterns and
    a superior mental conditioning. Finding such a mentor at the
    formative stage is probably the best thing to happen to anybody.

-   An effective strategy to find mentors is to shadow people on Linked
    in and learn from their profiles and activity. Some of them may be
    willing to connect and invest time in mentoring.

-   Another possibility to find like minded people and mentors would be
    to join the communities of on-line courses, like Datacamp and
    Dataquest, who have lively channels in Slack for paid members.

### Getting back up after a fall

Everybody breaks. Just as the promising student in Whiplash breaks. But
the champions among us rally, to stage a comeback and performance that
make history.

Regularly surpassing the level of deliberate knowledge of your own
performance, and thus improvement by exactly being able to measure your
performance and pinpoint mistakes. This point is portrayed in a very
interesting manner in Whiplash, where the teacher expects the student to
know exactly what mistake is being made.

### Be Great, not Good

Rejecting the \'Good\' or \'Good enough\' feedback from anybody. The
goal is to be *Great*, not good. The goal should be to strive to set the
precedent and not just follow a beaten track. The pinpoint focus should
be on progressive improvement to become the best, and that entails never
being satisfied and to be ruthless in rooting out flaws.

### Achieving Balance : mind + body + surroundings

Great performance is about that perfect balance between the body, mind
and environment to leverage the best result possible. I view the scene
where the student survives a car crash, just to reach a performance and
then not being able to perform, as a good example of overreaching,
without strengthening the core, and thus inviting instability.

### Go off the beaten track and Lose yourself

It was the ending of Whiplash that truly drove me to comprehend the
points so far. It is twisted, unexpected and led me to truly enjoy the
movie and appreciate that: despite the above points being reasonably
discernible - the human mind and nature is exceedingly complex.
Stability and reasoning are not the only keystones to the foundation of
greatness. There has to be a *healthy* mix of some form of abnormal
obsession thrown in, to make a stellar performance what it is. However,
can this be practically repeated on a regular basis?

### Learning velocity and Flow

There are several bodies of research work available today that can be
studied to get closer to consciously stimulating a great performance.
One such example is:

-   [Unlocking the Talent Code With Dan
    Coyle](https://unmistakablecreative.com/podcast/unlocking-the-talent-code-with-dan-coyle)
    on the Unmistakable Creatives podcast provides an insight in line
    with the points seen above, into what constitute outliers and
    performers. The interesting concept of \'Learning velocity\' is
    explained by Dan with a lucid example. It is surmised that progress
    and maximum learning to become better occurs *at* the boundary line
    dividing what we know at the moment, and the unknown skills that
    beckon.

That point sems to be an amalgamation of several factors, that are
typically present when someone is in \'flow\'. Perhaps this flow can be
described as a heightened sense of what is, and what should be and the
energy to strive and achieve what should be.. It certainly does feel
logical to think that we become better by pushing that boundary.

[TODO]{.todo .TODO} A history of my websites {#a-history-of-my-websites}
--------------------------------------------

This blog post enumerates my journey in creating a web presence. Most of
this was written long ago as a summary to myself, and is now updated to
October 2019.

### v5.0 - Discovered **Jekyll** recently (05/2015).

[Jekyll Now](http://www.jekyllnow.com) is *the* thing to check out!

-   Hosting on Github is pretty easy and very comfortable to develop an
    efficient workflow.
-   Some of the websites achieved by people are really superb (See
    link). And everything being open source - I can actually learn how
    they\'ve created their websites. This may raise issues of privacy.
    However, I suppose you can clearly specify what\'s personal and
    needs permission (like your blog notes). Git hub after all is meant
    for sharing code easily.
-   Though I\'m not a web developer by profession - my interest in being
    able to setup a customised platform to express myself - makes Jekyll
    and [Octopress](http://octopress.org) very interesting. In fact
    these two are called blogs for Hackers ! Very cool :)

### v4.0 of my website was set up using **Google Sites** in 2015 -

[link](https://sites.google.com/site/shreyasragavan/). There seemed to
be a lot of improvement in Google sites, since I tried it last somewhere
in 2012. However, there are limitations like a very small space
allotment (100 MB) and it also appears that it\'s complicated to get a
custom domain name. The themes are also quite lacking, and the blog
facility is very basic, called Announcements (though it does serve the
purpose).

-   I\'d recommend Google Sites over Weebly at the moment to very
    quickly set up a website and blog. The integration with Google
    services is pretty good. Just remember the 100MB space limit !

### v3.0 of my website was created created using **Wordpress with custom hosting**.

-   I found this relatively complicated to maintain and develop further.
    Of course there are plugins to achieve more or less anything that
    you need in Wordpress. I\'d even purchased the Ultimatum Framework
    to be able to construct page layouts. But my workflow was far from
    streamlined in the limited time available.
-   At this point, I formulated some basic guidelines for myself in this
    quest of setting up a web presence and web site: 1. It must be easy
    to maintain and deploy and shouldn\'t be costly. 1. The website
    should have a minimal, sophisticated (eventually) design that
    focusses on content. It should also load very quickly. 1. Must be
    possible to work on blog articles anywhere, anytime the inspiration
    strikes. 1. Must be easy to add interesting links and to continue my
    collation of interesting web resources. 1. The possibility must
    exist to easily transfer the existing posts and website without much
    work to another site. In short - it should be future proof. 1. In
    this context - markdown and text files are the best things I\'ve
    discovered in 2015. 1. Recently I\'ve combined the above with
    Jekyll, a static website generator, which is really fantastic, and
    seems to encompass all that I\'ve written above.

### v2.0 was done using **Weebly**, and is still the link I use often

(started 2012) -[Link](http://cfdrevolutions.weebly.com).

-   Weebly is certainly very very easy to use and I\'ve had no
    complaints whatsoever with their service/performance, despite being
    a free user. I\'d recommend Weebly for anybody who wants to set up a
    website quickly. In my opinion - their free package was the best at
    the time.
-   At this point, I\'d started using Storify (and have continued till
    date)

### v1.0 created using **Tiddlywiki**, (started 2011) -

[Link](http://cfdrev.tiddlyspot.com) The original goal was to find a
common platform and efficient technique to blog and collate good quality
online resources easily.

-   There are still several advantages and novelties in using Tiddlers.
    It can also be converted to some sort of a intranet of
    blogs/information in an organisation and there are simply many
    possibilities. The entire website can be downloaded in a single html
    file and is thus automatically portable.

[^1]: Articles on using Yasnippet: --- [Using Emacs Yasnippet against
    repetitive boileplate code](http://blog.refu.co/?p=1355) \|\|
    [Tweaking Emacs
    Yasnippet](https://jpace.wordpress.com/2012/10/20/tweaking-emacs-snippets/)
    \|\| [Expanding
    snippets](https://joaotavora.github.io/yasnippet/snippet-expansion.html)

[^2]: Links to using R with Emacs: [Using R with Emacs and
    ESS](https://www.r-bloggers.com/using-r-with-emacs-and-ess/) \|\|
    [Using R with Emacs](https://lucidmanager.org/using-r-with-emacs/)
    \|\| [Tips from R Coders who use
    ESS](https://www.reddit.com/r/emacs/comments/8gr6jt/looking_for_tips_from_r_coders_who_use_ess/)
    \|\| [Why I use Emacs for R
    programming](https://thescientificshrimper.wordpress.com/2018/12/12/soapbox-rant-why-i-use-emacs-for-r-programming/)

[^3]: See this [article of a non-technical user\'s
    experiment](http://rss.slashdot.org/~r/Slashdot/slashdot/~3/7iykh9HdS5U/i-stopped-using-a-computer-mouse-for-a-week-and-it-was-amazing)
    with not using the mouse, reporting significant gains in speed and
    productivity. I\'ve experienced this myself after gaining basic
    proficiency in moving around Emacs.

[^4]: To some extent, it is also possible that launchers like the Alfred
    app could be set or programmed to search in particular locations.
    This is a less *hacky* and still a functional option for Mac users.

[^5]: Fastmail allows for a variety of interesting features like
    aliases, easy email transfer (from a different email provider like
    Gmail or MSN), responsive technical support, and many more aspects,
    and much more. They have their own implementation of the IMAP
    protocol, [called
    JMAP](https://www.fastmail.com/help/guides/interfaceupdate-2018.html#what-is-jmap),
    which is significantly faster.

[^6]: While there are many advantages in Gmail and many swear by it\'s
    search capabilities - it is worth noting that Fastmail\'s ad-free
    interface and search just feels a lot quicker than Gmail, and I can
    find my way around the settings better than I used to with Gmail.

[^7]: You may be surprised to see the ease in browsing a good number of
    websites on a text based web browser. Besides the added advantage of
    being within Emacs - a surprising number of websites can be viewed
    functionally on w3m. It works fine for quick searches on Google
    (which like anything else, can be done within a few key strokes in
    Emacs).

[^8]: In [Scimax](https://github.com/jkitchin/scimax) - it is possible
    to quickly start a new project using `M-x
    nb-new`, which creates a sub-folder in the specified projects folder
    and creates and opens a readme.org file for the project.

[^9]: The option `C-u-cl` is a messy way to quickly get the full file
    name path, the resulting path will need to be modified slightly.

[^10]: It is worth noting that a bunch of additional HTML blocks and
    hyperlinks are inserted via the above export procedure. It should be
    possible to add some hooks to clean up the org file after the export
    from pandoc.

[^11]: Sometimes, this procedure has to be set specifically. Some good
    discussions on SO :
    [link1](https://stackoverflow.com/questions/2081577/setting-emacs-split-to-horizontal),
    [link2](https://stackoverflow.com/questions/7997590/how-to-change-the-default-split-screen-direction).
    However, at times horizontal splitting is useful. Therefore, I would
    rather not set a 0 width-threshold enabling only vertical splitting.

    ``` {.commonlisp org-language="lisp"}
    (setq split-width-threshold 75)
    (setq split-height-threshold nil)
    ```

[^12]: `C-x` essentially means Control + x. `M-x` or Meta-x is Alt + x

[^13]: The browse-kill-ring package can be installed via MELPA. (`M-x`
    install package)

[^14]: The latest version of IAWriter has a truck load of features and
    advantages over over the Classic version. I did consider purchasing
    it, but Emacs won the day. Nevertheless, as a plain vanilla writing
    app - IAWriter offers much right out of the box.

[^15]: This is my first animated gif in a blog post! It was tricky! I
    used the free [GIPHY capture
    app](https://itunes.apple.com/us/app/giphy-capture-the-gif-maker/id668208984?mt=12)
    on the Mac store.

[^16]: 

[^17]: IRC by the way is a treasure trove for particular areas of tech
    and interfacing with serious developers. I\'m currently using the
    IRC client Weechat, which can also connect to Slack by the way. Yes.
    Slack on the terminal. It is surprisingly functional and fast, but
    yes, it does take a bit of maneuvering.

[^18]: While Org mode is optimised for text, it is possible to attach
    any kind of file to a \'heading\', and use interleave and other
    techniques to browse and annotate PDF\'s. The possibilities are too
    numerous to be covered in a blog post or a single google search.

[^19]: On iOS - I\'ve found [Drafts](http://agiletortoise.com/drafts/)
    is a great app for writing fast and appending the notes to an org
    file, which can be refiled later, using emacs. One problem I\'m yet
    to resolve is that appending to an org file in dropbox, requires a
    network/internet connection. There should be a way to deal with
    situations without handy internet available.

[^20]: Nixnote is one solution. I\'ve seen it in action and it is
    useful, and probably even closer to DEVONthink. However, I could
    never get it working in Arch Linux reliably.

[^21]: It is probably worth noting that Feedly pro has several 3rd party
    integrations available out of the box including Evernote.