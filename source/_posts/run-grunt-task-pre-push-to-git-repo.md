---
title: Gruntjs tasks and git pre-push
date: 2013-07-23 01:00:00
tags:
---
<p>Oftentimes I am working on a <a href='http://gruntjs.com/'>gruntjs</a>-based project and have found it useful to run my default grunt task prior to pushing my code to my remote git repository. My default grunt task typically runs a jshint task, a qunit task, a concat task, and an uglify task &#8211; all of which you want executing without errors before your code makes it out into the wild.</p>

<p>And, luckily, git has a way to automatically run arbitrary bash code at various places in your git workflow using what&#8217;s called <a href='http://git-scm.com/book/en/Customizing-Git-Git-Hooks'>git hooks</a>.</p>

<h2 id='make_the_hook'>Make the hook</h2>

<p>Fire up your terminal and run the following commands in the root of your repository</p>

<pre class='bash'><code>touch .git/hooks/pre-push      # create the file; hooks directory location may vary &#x000A;chmod a+x .git/hooks/pre-push  # make it executable</code></pre>

<p>Next, place the following code inside the <code>pre-push</code> file and paste in the following code:</p>

<pre class='bash'><code>#!/bin/sh&#x000A;&#x000A;# check for how many uncommitted changes we have&#x000A;# stash changes&#x000A;# run grunt task &#x000A;# restore stashed files if anything was stashed&#x000A;# exit with error if grunt fails&#x000A;&#x000A;NAME=$(git branch | grep &#39;*&#39; | sed &#39;s/* //&#39;)&#x000A;&#x000A;echo &quot;Running pre-push hook on: &quot; $NAME&#x000A;&#x000A;# don&#39;t run on rebase&#x000A;if [ $NAME != &#39;(no branch)&#39; ]&#x000A;then&#x000A;  &#x000A;  CHANGES=$(git diff --numstat | wc -l)&#x000A;  CHANGES_CACHED=$(git diff --cached --numstat | wc -l)&#x000A;  TOTAL_CHANGES=$(($CHANGES + $CHANGES_CACHED))&#x000A;&#x000A;  git stash -k   # the &quot;-k&quot; makes git stash all changes, staged &amp; unstaged &#x000A;  grunt &#x000A;&#x000A;  RETVAL=$?&#x000A;&#x000A;  if [ $TOTAL_CHANGES -ne &quot;0&quot; ]&#x000A;  then&#x000A;    echo &quot;Popping&quot; $TOTAL_CHANGES &quot;changes off the stack...&quot;&#x000A;    git stash pop -q&#x000A;  fi      &#x000A;&#x000A;  if [ $RETVAL -ne 0 ] &#x000A;  then&#x000A;    echo &quot;Grunt task failed, exiting...&quot;&#x000A;    exit 1&#x000A;  fi&#x000A;&#x000A;  echo &quot;Complete.&quot;&#x000A;fi</code></pre>

<p>The code here is actually pretty straight-forward: any time we run <code>git push</code>, the <code>pre-push</code> script gets executed. The script counts the amount of uncommitted changes, stashes the changes, runs the default grunt task and if the grunt task fails, the stash is popped and we <code>exit 1</code> which tells git to fail the <code>push</code> command. If the grunt task does not fail, the push continues on.</p>

<p>What are your thoughts? Let me know.</p>
