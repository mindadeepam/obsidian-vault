
#### vscode failed to connect ssh - Failed to parse remote port from server output
[link](https://stackoverflow.com/questions/55979701/how-do-i-resolve-failed-to-parse-remote-port-from-server)
vscode ne kuch lock files create kardi hongi. ssh using terminal instead of vscode(ssh something...), then rm the vscode-server using 
```bash
rm -rf ~/.vscode-server/bin/**
```

#vscode


#### git push error - missing credentials
```
Missing or invalid credentials.
Error: connect ECONNREFUSED /run/user/337388563/vscode-git-f0e90613a7.sock
    at PipeConnectWrap.afterConnect [as oncomplete] (node:net:1595:16) {
  errno: -111,
  code: 'ECONNREFUSED',
  syscall: 'connect',
  address: '/run/user/337388563/vscode-git-f0e90613a7.sock'
}
Missing or invalid credentials.
Error: connect ECONNREFUSED /run/user/337388563/vscode-git-f0e90613a7.sock
    at PipeConnectWrap.afterConnect [as oncomplete] (node:net:1595:16) {
  errno: -111,
  code: 'ECONNREFUSED',
  syscall: 'connect',
  address: '/run/user/337388563/vscode-git-f0e90613a7.sock'
}
remote: Invalid username or password.
fatal: Authentication failed for 'https://github.com/FarMart-Tech/data_pipelines.git/'
```

just reload the vscode. [reference](https://chatgpt.com/share/bb1e82b9-c083-4a45-83c0-b24bba6db22a) 





### OSError cublas (symbol cublasLtGetStatusString version libcublasLt.so.11 not defined in file libcublasLt.so.11 with link time reference)

When installing torch in conda environments, sometimes we get an error like mentioned about. This happens because torch tries to install CUDAtoolkit which conflicts with the existing toolkit in your VM.

One possible solution to try out is to uninstall the toolkit installed by torch

```
pip uninstall nvidia_cublas_cu11
```

#Torch 

### conda not working in fish

Run `conda init fish` [reference](https://stackoverflow.com/a/58760411) 


### gcloud serviceunavailable error 

```
ServiceUnavailable: 503 Getting metadata from plugin failed with error: ('invalid_grant: Bad Request', {'error': 'invalid_grant', 'error_description': 'Bad Request'})
```
 
 run: 
`gcloud auth application-default login`

#gcp


### paste as plain text

Paste with Ctrl+Shift+V (or Option + Cmd + Shift + V on Mac). It will paste in plain-text, rather than with formatting. This is not an Obsidian thing. Pasting as plain-text is a great shortcut to know for all apps - example Excel or Google Sheets.