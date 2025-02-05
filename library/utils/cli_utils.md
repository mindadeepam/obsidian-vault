
Since mac by default has zsh, these commands are mostly zsh specific. 

`See more about the difference between zsh and bash`
	**How is bash different from zsh from a user perspective?**
		From a user perspective, Bash and Zsh are both popular Unix shell programs that provide a command-line interface for interacting with the operating system. While they share many similarities, there are a few notable differences between the two: 
		1. **Customizability:** Zsh is known for its high level of customization options. It offers extensive configuration capabilities, including advanced completion features, customizable prompts, and plugin frameworks like Oh My Zsh. Bash, on the other hand, has fewer customization options by default. 
		2. **Tab completion:** Zsh provides more advanced tab completion capabilities compared to Bash. It can autocomplete not only command names but also file paths, options, and arguments. Zsh's completion system is highly configurable and can even suggest completions based on historical usage patterns. 
		3. **Error correction:** Zsh includes a built-in feature called "command correction" that can suggest corrections for mistyped or misspelled commands. It offers options to automatically fix common mistakes or prompt the user for corrections. Bash does not have this built-in functionality. 
		4. **Array handling:** Zsh has better support for arrays compared to Bash. It provides more robust syntax and functionality for working with arrays, including associative arrays. In Bash, arrays are less versatile and lack some advanced features. 
		5. **Prompt customization:** While both shells allow prompt customization, Zsh offers more flexibility and advanced options for configuring your shell prompt. It supports a wide range of escape sequences and prompt expansion options, allowing users to create highly customized and informative prompts. 
		6. **Compatibility:** Bash is the default shell in many Unix-like systems and is widely used in scripting. It has been around for a long time and is highly compatible with POSIX shell standards. Zsh, while also compatible with POSIX, introduces some non-standard features, which may cause compatibility issues with scripts specifically written for Bash. It's worth noting that Bash has a larger user base and more extensive documentation due to its widespread usage. Zsh, on the other hand, has gained popularity among power users and developers who value its advanced features and customization options. Ultimately, the choice between Bash and Zsh often comes down to personal preference and specific use cases.  
	**Are basic commands different for them?**
		The basic commands in Bash and Zsh are largely the same, as they both adhere to common Unix shell conventions. Most common commands, such as `ls`, `cd`, `cp`, `mv`, `rm`, `mkdir`, `cat`, `echo`, and `grep`, work similarly in both shells. However, there are some minor differences in behavior and additional features that may vary between Bash and Zsh. Here are a few examples:  
		1. **Path expansion:** Zsh has some additional options for path expansion. For instance, you can use the `*` wildcard to match files and directories recursively.  
		2. **Command substitution:** Both shells support command substitution using backticks (\`) or the` $()` syntax. However, Zsh also allows you to use the `=(command)` syntax for process substitution, which can be useful in certain scenarios. 
		3. **History search:** Zsh provides more advanced history search capabilities compared to Bash. In Zsh, you can use keyboard shortcuts like `Ctrl+R` to search through your command history incrementally. 
		4. **Extended globbing:** Zsh supports extended globbing, which provides additional pattern matching options beyond what Bash offers. For example, you can use the `(pattern)` syntax to match zero or more occurrences of a pattern. 
		5. **Command line editing:** Zsh provides more advanced command line editing features, including better cursor movement, history navigation, and line editing shortcuts compared to Bash. It's important to note that while Zsh offers additional features, it strives to maintain compatibility with Bash, so most basic commands and scripts that work in Bash should also work in Zsh without issues. If you are already familiar with Bash and considering switching to Zsh, you will likely find that most of your existing knowledge and command usage will transfer seamlessly. However, you may also discover new features and capabilities in Zsh that can enhance your command line experience. - *other resources* - [https://apple.stackexchange.com/a/361957](https://apple.stackexchange.com/a/361957)

## Basic commands (by usage)

- ***find out OS of system(zsh,bash)***
    The **uname** command writes the name of the operating system implementation to standard output.  When options are specified, strings representing one or more system characteristics are written to standard output. 
    `uname -a` 
    
    `>> Darwin ITHELPDESKs-MacBook-Air.local 22.5.0 Darwin Kernel Version 22.5.0: Mon Apr 24 20:53:44 PDT 2023; root:xnu-8796.121.2~5/RELEASE_ARM64_T8103 arm64`
    
- **move source-folder inside target-folder**
    `mv source/ target/`
    
    `resulting dir structure:` `target/source/`
    
    note - this does not move contents of source to target, it moves entire the folder itself. 
    
    *to move contents of source to target,*
    `mv source/* target/`
    
    `resulting directories:` `target/<contents of source>`, `source/` 
    
-  *find process running at port 8080*
    `lsof -i:8080` : shows pid
	`kill -9 pid`
    
- ***get file size in dir***
	`du -h --max-depth=2 path_to_dir`

- **get linux distribution**
	  **`lsb_release -a`

- **reset terminal if it goes wonky** - `reset`
  (The `clear` command clears the terminal screen only while when talk about `reset` command, it not only clear the screen but also resets the terminal's settings to their default values.)
  
- **Chmod once and for all** - `chmod 666 filename`
	[Read this chat](https://chatgpt.com/share/80baf288-5314-456a-bbf5-f402a14ce297) 
	(r=4, w=2, x=1) sum them to get owner-group-others permissions

- use & to run a job in the background
  

- find a file in file directory: `find . -name results`
- find free space, linux: `free -h`
---

## Other commands


### 1. How to run a command in background, s.t. it keeps running even if we shut down the terminal?


```
# common commands 

screen -S <screen-name> : to start a screen session  
screen -ls : to view open sessions  
ctrl A+D : detach from a session  
screen -r <screen-name> : to resume a detached session  
screen -XS <screen-name> quit : to kill a detached screen session
```

*use-case - run instance in background after tuneling*

To keep a command running even if the terminal shuts down, you can use a terminal multiplexer like `tmux` or `screen`. Here's how you can achieve it using `tmux`: 1. Install `tmux` if it's not already installed. You can install it using your package manager. For example, on Ubuntu, you can run `sudo apt-get install tmux`. 2. Open your terminal and start a new `tmux` session by running the command `tmux`. 3. Inside the `tmux` session, run your desired command: ``` PROJECT_ID='data-science' export ZONE='us-west1-b' export INSTANCE_NAME='deepam-minda' gcloud compute ssh --project  $PROJECT_ID --zone $ZONE \\ $INSTANCE_NAME -- -L 8080:localhost:8080 ``` 1. After starting the command, you can detach from the `tmux` session by pressing `Ctrl+b` and then `d` (Ctrl+b, d). This will return you to your regular terminal session without stopping the running command. 2. You can now close your terminal, and the `tmux` session with the running command will continue to run in the background. When you want to reattach to the `tmux` session and check the status of the command, you can open a new terminal and run the command `tmux attach`. This will bring you back to the `tmux` session where you can see the output of the running command. Using `tmux` or a similar terminal multiplexer allows you to keep commands running even if the terminal or SSH session is closed.
To keep a command running even if the terminal shuts down, you can use a terminal multiplexer like `tmux` or `screen`. Here's how you can achieve it using `tmux`:
    
1. Install `tmux` if it's not already installed. You can install it using your package manager. For example, on Ubuntu, you can run `sudo apt-get install tmux`.
2. Open your terminal and start a new `tmux` session by running the command `tmux`.
3. Inside the `tmux` session, run your desired command:


```
PROJECT_ID='data-science'
export ZONE='us-west1-b'
export INSTANCE_NAME='deepam-minda'
gcloud compute ssh --project $PROJECT_ID --zone $ZONE \\
$INSTANCE_NAME -- -L 8080:localhost:8080   
```
    
1. After starting the command, you can detach from the `tmux` session by pressing `Ctrl+b` and then `d` (Ctrl+b, d). This will return you to your regular terminal session without stopping the running command.
2. You can now close your terminal, and the `tmux` session with the running command will continue to run in the background.

When you want to reattach to the `tmux` session and check the status of the command, you can open a new terminal and run the command `tmux attach`. This will bring you back to the `tmux` session where you can see the output of the running command.

Using `tmux` or a similar terminal multiplexer allows you to keep commands running even if the terminal or SSH session is closed.

For macOS, you can use a terminal multiplexer called `screen`. Here's how you can achieve the same result using `screen`:

1. Open your terminal.

2. Start a new `screen` session by running the command `screen`.

3. Inside the `screen` session, run your desired command: 

    ```shell
    PROJECT_ID='data-science'
    export ZONE='us-west1-b'
    export INSTANCE_NAME='deepam-minda'
    gcloud compute ssh --project $PROJECT_ID --zone $ZONE \
    $INSTANCE_NAME -- -L 8080:localhost:8080
    ```
    
    4. After starting the command, you can detach from the `screen` session by pressing `Ctrl+a` followed by `d` (Ctrl+a, d). This will return you to your regular terminal session without stopping the running command.
    
    5. You can now close your terminal, and the `screen` session with the running command will continue to run in the background.
    
When you want to reattach to the `screen` session and check the status of the command, you can open a new terminal and run the command `screen -r`. This will reattach to the existing `screen` session, where you can see the output of the running command.

Using `screen` allows you to keep commands running even if the terminal or SSH session is closed.




### 2. gsutil

- copy a dir recursively from gcp to local
  `gsutil cp -r [OPTIONS] SOURCE DESTINATION`
  add `-m` for parallel download
  add `-P` for viewing progress
  
  eg: `gsutil -m cp -r -P  gs://fmt-ds-bucket/dev/grain_classification_model_data/imgs/wheat . 

### 3. Launching a process from a running process; Differences Between os.execvp, subprocess.run, and os.system.




1. os.execvp:
•	Behavior: Replaces the current process with the new one. The current Python process is terminated, and the uvicorn process takes its place. Control does not return to the Python script after execution.
•	Use Case: Use when you want to completely replace the current Python process with the new program.
•	Environment Variables: The new process inherits the current environment variables unless explicitly modified before calling os.execvp.
2. subprocess.run:
•	Behavior: Spawns a new child process to run the command. The parent Python process continues to exist and can wait for the child process to complete or perform other tasks simultaneously.
•	Use Case: Use when you want to run a subprocess and still retain control of the current Python process.
•	Environment Variables: By default, the subprocess inherits the current environment variables. You can customize them using the env parameter.
3.	os.system:
•	Behavior: Executes the command in a subshell. The current Python process remains active and waits for the shell command to complete.
•	Use Case: Use for quick shell commands without needing advanced control over the process.
•	Environment Variables: The command inherits the current environment variables unless altered before the os.system call.


Recommendations:
	•	Use os.execvp when you want to replace the Python script entirely with the new process.
	•	Use subprocess.run for more robust control over the subprocess.
	•	Use os.system for simple, quick shell execution tasks.