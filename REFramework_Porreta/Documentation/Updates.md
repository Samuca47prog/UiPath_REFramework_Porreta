### KillAllProcess.xaml

#### Implementation
KillAllProcess is used to kill all process defined in key "ProcessToKill" of Config, which is defined via Asset.

KillProcess.xaml is a sub workflow implemented to kill a list of given process names.
It uses Kill Process activity surrounded by a try catch to retrieve exceptions in case of failute to kill process.

#### How to use?
- If you want to **kill any given processes**, use KillProcess.xaml passing the list of process names as ```{"Notepad", "mspaint"}```
- If you want to **kill all the processes**, make sure your processes names are ```;``` separated at ProcessToKill asset and invoke KillAllProcesses.xaml  
Asset ```ProcessToKill``` name is defined in Config Assets sheet.