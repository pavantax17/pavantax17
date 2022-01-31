Set-StrictMode -Version 4.0
$ErrorActionPreference = "Stop"
##Import-module "sqlps" -Force
$servername = ""
$database = ""
$email_to = "" # modify here to address#
$email_from = ""
$Logs_Collection_Path = Join-path -path "D:\DBA" -Childpath "Logs"
$Data_Collection_Path = Join-path -path "D:\DBA" -Childpath "Data"
$Global:Date = (Get-Date).ToString('MMddyyyy_HHmmss')
$currdate=(Get-date).ToString('yyyyMMdd')
$subjectdate=Get-date -Format 'yyyy-MM-dd'
$Destination="\\vrisi01\Shared$\Corporate\AccessReviewSystem"
$Scriptlogfile = Join-Path $Logs_Collection_Path -ChildPath "Script_to_send_Monthly_CADA_DB_ARS_$Date.txt"
$datafile = Join-path -path $Data_Collection_Path -Childpath "CADA_DB_ARS_$currdate.csv"
$checkfile=join-path $Destination -ChildPath "CADA_DB_ARS_$currdate.csv"
$countlogfile=join-path $Destination -ChildPath "CADA_DB_ARS_$currdate.txt"
$Comment = "[Info]:Logging to the file $Scriptlogfile"



write-verbose $Comment -verbose
$Comment | Out-file $Scriptlogfile -append
Try{
$Comment = "[Info]:starting the data load"
write-verbose $Comment -verbose
$Comment | Out-file $Scriptlogfile -append



$query = @"



--queryhere--
"@



##Invoke-Sqlcmd -ServerInstance $servername -Database $database -Query $query | Export-Csv -Path $datafile -NoTypeInformation -Verbose



$output = Invoke-Sqlcmd -ServerInstance $servername -Database $database -Query $query
$output | Export-Csv -Path $datafile -NoTypeInformation -Verbose
##Treasury_ARS_EVIDENCE_20210329
$datetime=Get-date
$RecordCount=$output.Count
$Comment ="Report Generated on :$datetime Number of records read:$RecordCount"
$Comment |Out-File $Countlogfile

$Comment = "[Info]:completed the data load"
write-verbose $Comment -verbose
$Comment | Out-file $Scriptlogfile -append



$Comment = "[Info]:Copying $datafile to the location $Destination"
write-verbose $Comment -verbose
$Comment | Out-file $Scriptlogfile -append



Copy-Item $datafile -Destination $Destination
If ( Test-Path $checkfile){
$Comment = "[Info]:$checkfile Exist"
write-verbose $Comment -verbose
$Comment | Out-file $Scriptlogfile -append
}



$Comment = "[Info]:Sending email"
write-verbose $Comment -verbose
$Comment | Out-file $Scriptlogfile -append
$smtp = "smtpgw.chec.local"

$to = $email_to
$from = $email_from
$body=@"
Hi Team,



Find attached Monthly CADA file report.



Regards
DBA Team
"@
$subject = " Monthly CADA File $subjectdate"



Send-MailMessage -from $from -to $to -SmtpServer $smtp -Subject $subject -Body $body -Attachments $datafile -Verbose
}
Catch{
$Comment = "[Error]:While Running Script_to_send_Monthly CADA File.ps1"
write-verbose $Comment -verbose
$Comment | Out-file $Scriptlogfile -append
$_ | Out-file $Scriptlogfile -append



}
