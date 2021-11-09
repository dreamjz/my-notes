# Spf13-vim

## Plugins

spf13-vim contains a curated set of popular vim plugins, colors, snippets and syntaxes. Great care has been made to ensure that these plugins play well together and have optimal configuration.

### Adding new plugins 

Create `~/.vimrc.boundles.local` for any additional bundles.

To add a new bundle, just add one line for each bundle you want to install. The line should start with the word “Bundle” followed by a string of either the vim.org project name or the githubusername/githubprojectname. For example, the github project [spf13/vim-colors](https://github.com/spf13/vim-colors) can be added with the following command

```
echo Bundle \'spf13/vim-colors\' >> ~/.vimrc.bundles.local
```

Once new plugins are added, they have to be installed.

```
    vim +BundleInstall! +BundleClean +q
```

### Removing (disabling) an included plugin

Create `~/.vimrc.local` if it doesn't already exist.

Add the UnBundle command to this line. It takes the same input as the Bundle line, so simply copy the line you want to disable and add 'Un' to the beginning.

For example, disabling the 'AutoClose' and 'scrooloose/syntastic' plugins

```
    echo UnBundle \'AutoClose\' >> ~/.vimrc.bundles.local
    echo UnBundle \'scrooloose/syntastic\' >> ~/.vimrc.bundles.local
```

**Remember to run ':BundleClean!' after this to remove the existing directories**

Here are a few of the plugins:

### [YouCompleteMe](https://github.com/Valloric/YouCompleteMe)

YouCompleteMe is another amazing completion engine. It is slightly more involved to set up as it contains a binary component that the user needs to compile before it will work. As a result of this however it is very fast.

To enable YouCompleteMe add `youcompleteme` to your list of groups by overriding it in your `.vimrc.before.local` like so: `let g:spf13_bundle_groups=['general', 'programming', 'misc', 'scala', 'youcompleteme']` This is just an example. Remember to choose the other groups you want here.

Once you have done this you will need to get Vundle to grab the latest code from git. You can do this by calling `:BundleInstall!`. You should see YouCompleteMe in the list.

You will now have the code in your bundles directory and can proceed to compile the core. Change to the directory it has been downloaded to. If you have a vanilla install then `cd ~/.spf13-vim-3/.vim/bundle/YouCompleteMe/` should do the trick. You should see a file in this directory called install.sh. There are a few options to consider before running the installer:

- Do you want clang support (if you don't know what this is then you likely don't need it)?
  - Do you want to link against a local libclang or have the installer download the latest for you?
- Do you want support for c# via the omnisharp server?

The plugin is well documented on the site linked above. Be sure to give that a read and make sure you understand the options you require.

For java users wanting to use eclim be sure to add `let g:EclimCompletionMethod = 'omnifunc'` to your .vimrc.local.

