artifact-manager
================

A Python script to manage a repository of project artifacts, possibly
versioned.  Each repository contains a list of "branches"; each branch
is defined by a list of artifacts (which may vary or repeat
individually from branch to branch). Location info for each artifact
in the project tree is preserved.

It has been designed to play nicely with project hosted in a PDIHub
Git repository (i.e. it manages the files that cannot go into PDIHub
since they are binary artifacts), but can also be used independently.

It accepts the following operations:

 * __list__  List the artifacts stored in a remote repo for a given branch

 * __check__ Compare the artifacts in the local tree with the remote repo

 * __download__  Fetch artifacts for the current branch from the repo.
     Only new & modified artifacts will be downloaded. They will be
     left in their defined places in the project tree.

 * __upload__  Upload all local artifacts to the repo, defining the set for
       the current branch. Only new/modified files will be uploaded

 * __purge__	  *future op*

 * __setoptions__ *future op*


Intended PDIHub workflow
------------------------

To manage the artifacts associated to a PDIHub project, the envisioned
steps are as follows:

* To download artifacts, after the Git project has been cloned
  into a local repo, and the desired branch has been checked out, just
  execute `artifact-manager download` over the base directory of the
  project. The artifacts corresponding to that branch of the project
  will then be downloaded (over HTTP).

* Whenever the project switches to another branch, executing again
  `artifact-manager download --delete-old` will connect to the
  artifact repository and update the artifacts in the local directory
  to match the new branch (the script is smart enough to download only
  the new or changed files, which comes handy if those files are large).

* For artifact upload, execute on the local folder the command
  `artifact-manager --base-url <dir> upload` [1]. It will collect all
  local artifacts and upload them to the server, labelling them as
  belonging to the current branch [2]

* If the local project changes to another branch, repeat the upload process
  and the local artifacts will be registered (and uploaded, if necessary) as 
  belonging to the new branch

* Whenever there is variation in the local artifacts, issuing the command  
  `artifact-manager --base-url <dir> upload --overwrite` will keep 
  the remote repository in sync with the local files. Note however that 
  the previous configuration for that branch will be lost: the server 
  keeps *only one version* of an artifact snapshot per branch


[1] Currently the only working R/W transport is a local folder, so in
order for this to work, the remote server must be locally mounted as a
network disk (a native SMB transport layer is in the works)

[2] The first time this command is executed on a project will create
the artifact repository for that project in the repository server.




Transports
----------

Available transports to connect to the remote repository are:

* HTTP for read-only operations (list, check, download)
* Local folder for upload operations (a remote repo can be uploaded to if its folder is mounted as a network disk)
* SMB (_in the works_) for upload operations


Repository specification
------------------------

A repository is defined on top of a transport layer (each transport
endpoint can contain a number of repositories) with a string. On top
of that, a branch is also an arbitrary string (usually with a path-like shape)

These parameters can also be automatically defined if the project tree
is a checked out Git repo. In this case Git is used to extract the
name of the artifact repository to use (will have the same name as the
git repo, minus the `.git` suffix) and the artifact branch (the
currently checked out branch). This selection can be overriden with
command line parameters.


In summmary, the parameters needed for operation that can be defined  are

* Local project tree: use a positional argument; else the current
  working directory will be used
* Repository name: use the `--repo-name <name>` option; else it will
  be taken from Git info
* Branch name: use the `--branch <name>` option; else it will be taken from 
  Git info

To be able to fetch Git info, the `git` command-line executable will
be tried first; if not available the ".git" directory will be searched.



Execution options
-----------------

Options defining the behaviour are:

* `--extensions` : a comma-separated list of the file extensions (with or
  without a leading period) that will be collected as artifacts
* `--min-size`: the minimum size of an artifact file to be included in
  the list (0 for files of any size)

These are settable _per repository_: when the remote repository is
created (upon first artifact upload) the _extensions/min-size_
definitions at that time are stored as repository configuration, and
used henceforth. It can be overriden at runtime for a given execution.

Other command-line options are:

* --verbose <n>
* --dry-run
* --overwrite
* --delete-old


Requirements
------------

* Python 2.7 (probably also Python 2.6)

* The [pysmb](https://pypi.python.org/pypi/pysmb/1.1.5) Python module,
  only to use SMB transports for artifact upload
