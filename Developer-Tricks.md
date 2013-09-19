Note: this is a page of useful tricks for developers. 
## Mac 

### Git completion
https://github.com/git/git/tree/master/contrib/completion

* git-prompt.sh. This changes the prompt on the command line to show the current branch. Install by copying it to ~/, follow install instructions. For a cleaner prompt, replace the PS1 suggested with: 
`PS1='\w$(__git_ps1 " (%s)")> '`
The prompt will look like:
`~/stan (master)> `
where "~/stan" is the current directory, "(master)" indicates the current branch is the master branch.

* git-completition.*sh. Install this for auto-completion from the command line. It auto-completes git commands and git branches. For example, type `git checkout ` then hit tab twice. It should show the available branches.

### Aquamacs: single kill buffer
By default, aquamacs will has multiple kill buffers. This means that there is a copy and paste buffer by using command-c/x/v and there is a separate copy and paste buffer by using ctrl-w, alt-w, ctrl-y. This gets really confusing. Here's how to have a single kill buffer so copying from any Mac program will paste into emacs using ctrl-y or command-v.

1. Open ~/.emacs (or wherever else you're storing your preferences)
2. Add: `(setq x-select-enable-clipboard t)`

## Fixing Line Feeds and Tabs

Ant includes a handy task called <a href="http://ant.apache.org/manual/Tasks/fixcrlf.html">FixCRLF</a> that "Adjusts a text file to local conventions."  So you can set it to replace tabs with spaces and Windows line ends with unix ones.

## Emacs

### Auto-removing tabs

To make sure you never have tabs in your code files, you can use this in your ```.emacs``` file:

```
(defun java-mode-untabify ()
   (save-excursion
     (goto-char (point-min))
     (if (search-forward "t" nil t)
         (untabify (1- (point)) (point-max))))
   nil)

 (add-hook 'java-mode-hook
           '(lambda ()
              (make-local-variable 'write-contents-hooks)
              (add-hook 'write-contents-hooks 'java-mode-untabify)))

 (add-hook 'html-mode-hook
           '(lambda ()
              (make-local-variable 'write-contents-hooks)
              (add-hook 'write-contents-hooks 'java-mode-untabify)))

 (add-hook 'cpp-mode-hook
           '(lambda ()
              (make-local-variable 'write-contents-hooks)
              (add-hook 'write-contents-hooks 'java-mode-untabify)))

 (add-hook 'stan-mode-hook
           '(lambda ()
              (make-local-variable 'write-contents-hooks)
              (add-hook 'write-contents-hooks 'java-mode-untabify)))
```

You can rename --- the "Java" in the title is a holdover from where I first got the macros.
