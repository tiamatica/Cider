[parm]:title             = 'Cider Reference'
[parm]:numberHeaders     = 2 3 4 5 6
[parm]:leanpubExtensions = 1



# Cider's API --- Syntax Reference

Although the API functions are similar to their user command equivalents, they are not identical. Also, the API offers more functions, and for two user commands there are no corresponding functions in the API (`CreateProject` and `CloseProject`). 

While for the user commands the help provided by the `-?` syntax is sufficient, this document deals with the API.

Note that while `⎕SE._Cider` contains the Cider code , the API is available via `⎕SE.Cider`.

The API functions are ordered alphabetically.

## AddAlias

Takes two parameters via the right argument:

1. Fully qualified folder hosting a project
2. Alias name

Actions taken:

* If the alias is not already in use and the folder exists the function adds the definition to the file returned by `GetAliasFilename`. 

* If the alias is already in use and the folder exists the function asks for confirmation before overwriting the definition of that alias in the file returned by `GetAliasFilename`, effectively changing the alias.

Returns an empty vector in case of success and an error message otherwise.


## CloseProject

There is no API equivalent for the user command `]Cider.CloseProject`. Use `⎕SE.Link.Break` instead.

## CreateOpenParms

A monadic function that returns a namespace with default parameters required by the `OpenProject` function. The right argument is ignored.


```
 alias         ''      
 folder        ''  
 parent        '#' 
 projectSpace  ''  
 quietFlag     0 
 suppressInit    0 
```

This is useful for creating a namespace with the defaults, set required parameters (at least `folder` and `projectSpace`), make amendments and pass the namespace as argument to `OpenProject`.

## CreateProject

There is no API equivalent for the user command `]Cider.CreateProject`.

## DropAlias

Takes an alias and removes it.


## GetAliasFileContent

Niladic function that returns the contents of the file returned by the `GetCiderAliasFilename` function as an APL array:

```
      ⎕SE.Cider.GetAliasFileContent
 cider  C:/T/Projects/Dyalog/Cider 
```

Every record of that file contains two pieces of information: an alias and its associated path. The file might be empty.

## GetAliasFilename

Niladic function that returns the path to the file that is used by Cider to manage alias names and their paths.


## GetMyUCMDsFolder

Niladic function that returns the path to the `MyUCMDs/` folder.

Note that this folder is created under Windows by the Dyalog installation routine, but not on other platforms.


## GetCiderGlobalConfigFileContent

Niladic function that returns the contents of the file returned by the `GetCidersConfigFilename` function as a namespace with variables.

The file might not exist, or be empty. In either case the function returns `⍬`.

For the time being the file may define `ExecuteAfterProjectOpen`.


### ExecuteAfterProjectOpen

The user may specify a fully qualified function name, usually situated in `⎕SE`. This function will then be called after a project was opened by Cider. 

This can be used to, say, check the status of the project on GitLab or a similar platform, since Cider considers only GitHub, and requires the package [APLGit2](https://github.com/aplteam/APLGit2 "Link to APLGit2 on GitHub") for this.

This setting defines the same function for all your Cider projects which is why it is not part of the file `cider.config.template` but defined in Cider's global config file.

## GetCiderGlobalConfigFilename

Niladic function that returns the path to the file with Cider's global configuration file.

It's expected to be a folder `.cider` in the user's home directory on all platforms.

## ListOpenProjects

Requires a Boolean as right argument ("verbose").

With 0 it returns a matrix with two columns:

```
      ⎕SE.Cider.ListOpenProjects 0
 #.Cider  /path/to/Cider
```

With the verbose flag set it returns 4 columns:

```
      ⎕SE.Cider.ListOpenProjects 1
 #.Cider  /path/to/Cider  32  cider 
```
`[;1]` | Fully qualified project space
`[;2]` | Path the project was loaded from
`[;3]` | Number of objects belonging to the project
`[;4]` | Alias (if any)

## ListTatinPackages

Lists all dependencies installed in the Tatin installation folders, if any. 

For every install folder these pieces of information are listed:

* The full package ID
* A Boolean indicating whether the package is a principal one or just a dependency
* A URL where the package was loaded from

## OpenProject

Opening a project means carrying out the following actions:

1. Creating the `projectSpace` (namespace) in `parent` if it does not already exist

   From here on we refer to this as the _root of the project_.
2. Setting the system variables `⎕IO` and `⎕ML` in the root of the project
3. Bringing all code and variables into the root of the project
4. Loading all Tatin packages from the Tatin installation folders defined by `dependencies` and `dependencies_dev`
5. Injecting a namespace `CiderConfig` into the root of the project and populating it with the contents of the configuration file as an APL array
6. Adding a variable `HOME` to `CiderConfig` that carries a path to where the project was loaded from
7. Executing the project-specific function noted on `init`, usually to initialize the project
8. Executing a non-project-specific function defined in Cider's own configuration file

   This can be used for carrying out the same user-specific actions after a project was opened.

`OpenProject` requires on of the following two options as right argument:

* A character vector with either a path pointing to a folder that holds a project config file (`folder`) or an alias defining a project.

* A parameter space carrying appropriately named variables, typically created by calling `CreateOpenParms`

  Must contain a variable `folder` that is not empty.

All possible variables are discussed below, with `folder` being the first one because it is mandatory: without it Cider would not be able to locate a project config file. The remaining ones have appropriate defaults or are taken from the config file, and they are sorted alphabetically.

What's defined in the parameter space overwrites the value in the config file, though only temporarily.

The function returns a two-element vector:

* A Boolean, 1 indicating success
* A vector of character vectors with what got printed to the session as well

### folder (mandatory)

This must be one of:

* A path pointing to a folder that carries a file `cider.config`
* An alias like `[aliasname]`

Note that this must _not_ be empty but it _might_ be `./` representing the current directory.

### alias

The alias under which you might want to access the project in the future.

### checkPackageVersions

This is ignored in case...

* the project does not sport any Tatin installation folder
* [`importFlag`](# {style="color: red;"}) is 1; no check will take place then anyway

By default this is an empty numeric vector (`⍬`), meaning that the user will be asked whether she wants Cider to check all principal packages for later versions, and if there are any found, whether she wants to update those packages. If the project carries more than one package folder the second question is asked independently for each Tatin installation folder.

Instead one can set this parameter to these values:


| 0 | Do not check at all
| 1 | Check and report findings but prompt for updating
| 2 | Check and update without consulting the user


### parent

Defaults to `#` but might as be something like `⎕SE` or `#.Foo.Goo.Boo`. However, all namespaces listed must exist.


### projectSpace 

The name of the namespace the project is injected into. If this is empty it is going to be `#` or `⎕SE`, depending from where the function was called from.


### Flags


#### importFlag

Defaults to 0 meaning that the code is not imported but linked. 

Set this to 1 for Link importing the code without establishing a Link.

Note that this has implications on how Cider deals with Tatin packages, see there.


#### noPkgLoad

Defaults to 0, meaning that Cider will load Tatin packages by honoring the config file's `dependencies` and `dependencies_dev` parameters. However, there might be circumstances when you do not want packages to be loaded. This can be achieved by setting the `noPkgLoad` flag to 1.


#### quietFlag

Defaults to 0, meaning that the function prints messages to the session. If this is 1 then no messages are printed but what would have been printed is still returned as second element of the result of the function.


#### suppressLX

Defaults to 0, meaning that the projects initialisation function (if any) will be executed.

There might well be situations when you don't want this, therefore you may suppress the execution by setting this to 1.

An example might be an automated build process: that might need to open the project but without actually initialising it.

### watch

Defaults to "both", meaning that changing any APL objects in the workspaces will result in Link updating the corresponding file on disk, and any change on disk will be reflected by changing the workspace.

You may instead set this to "ns", which forces Link to write changes in the WS to disk but ignore changes on the file system.

You may also set this to "dir", when changes on the file system will be brought into the WS while changes in the WS are _not_ saved to disk.

Other settings of `watch` will result in an error.

A> ### `watch←'both'|'dir'`
A>
A> In order to detect changes on the file system Link uses a File System Watcher, something that is available only under Windows.
A> Link uses APL threads for this.
A>
A> When you trace through your code, or set a stop vector, and have "Pause on Error" in the session's "Threads" menu ticked, any handler associated with those threads will also stop.
A>
A> This imposes a danger because those handlers set a Hold under some circumstances, and depending on your actions this might result in a DEADLOCK: Dyalog would appear to hang until you issue a strong interrupt via the session's system menu item.


## ReadProjectConfigFile

This function takes a path to a Cider project as `⍵` and returns the contents of the config files as a namespace with variables.

`path` may or may not come the actual filename `cider.config'


## WriteProjectConfigFile

This function takes a path to a Cider project as `⍵` and a namespace with Cider config variables as `⍺` and writes it the Cider config file.

`path` may or may not come the actual filename `cider.config'


## Version

`Version` returns a three-element vector with these pieces of information:

1. The name, "Cider"
2. The version number

   This can be just `1.2.3`,  but it may be something like `1.2.3-beta-1+113`
3. The version date in international date format: YYYY-MM-DD
