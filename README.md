# windows连接nas的各类问题处理
## 一、Windows10/11在连 Nas 映射网络驱动器后，开机时显示红叉，但是双击后又能使用的解决方案。
### 1、新增文件，名字改成：MapDrives.ps1，保存在目录 C:\Scripts【没有需要新建一个】  
MapDrives.ps1的代码如下：
```
$i=3
while($True){
$error.clear()
$MappedDrives = Get-SmbMapping |where -property Status -Value Unavailable -EQ | select LocalPath,RemotePath
foreach( $MappedDrive in $MappedDrives)
{
try {
New-SmbMapping -LocalPath $MappedDrive.LocalPath -RemotePath $MappedDrive.RemotePath -Persistent $True
} catch {
Write-Host "Shared folder connection error: $MappedDrive.RemotePath to drive $MappedDrive.LocalPath"
}
}
$i = $i - 1
if($error.Count -eq 0 -Or $i -eq 0) {break}
Start-Sleep -Seconds 30
}
```
### 2、打开目录 C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp ，新增文件，名字改成: MapDrives.cmd
MapDrives.ps1的代码如下：
```
PowerShell -Command "Set-ExecutionPolicy -Scope CurrentUser Unrestricted" >> "%TEMP%\StartupLog.txt" 2>&1
PowerShell -File "%SystemDrive%\Scripts\MapDrives.ps1" >> "%TEMP%\StartupLog.txt" 2>&1
```
### 3、按win键，输入“任务计划程序” -> 打开
### 4、点击创建任务
#### 4.1、名称随便写，但必须要是英文的，否则会无法启动 - 点击“不管用户是否登录都要运行” - 点击“使用最高权限运行” - 勾选“隐藏” - 配置选“windows 10”  
#### 4.2、单击选项卡“操作” - 单击“新建” - 选择“启动程序” - 单击“浏览” - 找到刚刚保存的文件 "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\MapDrives.cmd"  
#### 4.3、单机选项卡“条件” - “只有在以下网络连接可用时才启动” - 选择“任何连接”  
### 5、单击“确定”，关闭管理页面，重启电脑大概率能解决问题。
>**特别注意：开机后有可能会弹出黑框执行命令，一定不要手动关闭！！！**
