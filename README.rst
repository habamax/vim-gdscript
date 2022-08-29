================================================================================
                        Vim Files For Godot Game Engine
================================================================================

Extracted from https://github.com/habamax/vim-godot to be included into Vim.


Additional setup you can have in your ``~./vim/after/gdscript.vim``:

.. code:: vim

  vim9script

  setlocal noet ts=4 sw=0

  var last_scene_run = ''

  # Silently execute OS command
  def Exe(cmd: string)
      var job_opts = {}
      if exists("$WSLENV")
          job_opts.cwd = "/mnt/c"
      endif
      job_start(cmd, job_opts)
  enddef

  # Run last scene
  def RunLast()
      if last_scene_run == ''
          echomsg "No scene was run yet!"
          return
      endif
      RunScene(last_scene_run)
  enddef

  # Run current scene
  def RunCurrent()
      RunScene(expand("%:r") .. '.tscn')
  enddef

  # Run arbitrary scene
  def RunScene(scene_name: string)
      if !exists('g:godot_executable')
          if executable('godot')
              g:godot_executable = 'godot'
          elseif executable('godot.exe')
              g:godot_executable = 'godot.exe'
          else
              echomsg 'Unable to find Godot executable, please specify g:godot_executable'
              return
          endif
      endif

      var godot_command = $'{g:godot_executable} {scene_name}'
      Exe(godot_command)
      last_scene_run = scene_name
  enddef


  nnoremap <buffer> <F5> <scriptcmd>RunScene("")<CR>
  nnoremap <buffer> <F6> <scriptcmd>RunCurrent()<CR>
  nnoremap <buffer> <F7> <scriptcmd>RunLast()<CR>


If you want to have FZF like scene selection, add more:

.. code:: vim

  import autoload 'popup.vim'

  def RunSelectedScene()
      var scenes = []
      if executable('fd')
          scenes = systemlist('fd --path-separator / --type f --hidden --follow --exclude .git --glob *.tscn')
      elseif executable('rg')
          scenes = systemlist('rg --path-separator / --files --hidden --glob !.git --glob *.tscn')
      else
          return
      endif
      popup.FilterMenu("Run scene", scenes,
          (res, key) => {
              RunScene(res.text)
          },
          (winid) => {
              win_execute(winid, 'syn match FilterMenuDirectorySubtle "^.*\(/\|\\\)"')
              hi def link FilterMenuDirectorySubtle Comment
          })
  enddef

  nnoremap <buffer> <space>r <scriptcmd>RunSelectedScene()<CR>

where ``popup.vim`` is an `autoloaded file`__ you could think of re-using.

__ https://github.com/habamax/.vim/blob/77bb48d6aa986705088b3400a122f250c660678e/autoload/popup.vim#L49-L192
