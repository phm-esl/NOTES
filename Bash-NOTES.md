# Upgrade Bash for MacOS

Modify the default shell to add the Bash-5.1.16 compiled from source.  This
involved appending the path to new Bash to the /etc/shells file, then
specifying the new Bash shell in the Terminal application, and adding
initialisation scripts to collect the $PATH from the programs in
$HOME/Software, and to set up the prompt.

Yet to do is Bash completion scripts for editing source files in Erlang
directory heirarchies, and Makefile targets.


    sudo su -
    cd /etc
    cp shells{,-ORIGINAL}
    nano shells

    diff shells{-ORIGINAL,}
    11a12,13
    > /Users/phm/Software/bash-5.1.16/bin/bash
    > 

* Terminal -> Preferences...
* Shells open with:
**   [*] Command (complete path)
**     [/Users/phm/Software/bash-5.1.16/bin/bash]

    cat $HOME/.bash_profile 
    if [ "${BASH-no}" != "no" ]; then
            [ -r $HOME/.bashrc ] && . $HOME/.bashrc
    fi

    cat ~/.bashrc
    . ~/Scripts/bash-environment


    cat ~/Scripts/bash-environment
    # Write history after each command
    _bash_history_append() {
        builtin history -a
    }

    _load_completion_file() {
    declare -ra Array=( $HOME/Software/bash-*/share/bash-completion/bash_completion )
    declare -r Complete="${Array[-1]}"
    if ! shopt -oq posix; then
      if [ -f "${Complete}" ]; then
        . "${Complete}"
      fi
    fi
    }

    _load_completion_file;

    export EDITOR=nano
    export VISUAL=nano

    export PROMPT_DIRTRIM=4
    export PROMPT_COMMAND=$'ERRCODE=$?;
    if [ $ERRCODE -ne 0 ];
    then ERRFLAG="\E[31;01merror = $ERRCODE\E[39;49;00m \n";
    else ERRFLAG=;fi;
    _bash_history_append; '
    export PS1=$'${ERRFLAG}\[\e]2;\u@\h \w\a\]\[\e[31m\]\
    `date +%Y-%m-%dT%H:%M:%S%:::z` \[\e[32m\]\u@\h\[\e[00m\]:\
    \[\e[01;34m\]\w\[\e[00m\]\n\!> '

    for p in $HOME/Software/*/bin/ $HOME/bin/ ;
    do [ -d "$p" ] && PATH="${p%/}:${PATH}" ;  
    done
    export PATH

    for p in $HOME/Software/*/{man,share/man}/ ;
    do [ -d "$p" ] && MANPATH="${p%/}:${MANPATH}" ;
    done
    export MANPATH
