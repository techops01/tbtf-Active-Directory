# Active-Directory trusts between two forests


0.Добавляем необходимые для работы порты в исключения входящих/исходящих соединений через Powershell на обеих доменах:
New-NetFirewallRule -DisplayName "Trust between domains" -Direction inbound -Profile Any -Action Allow -LocalPort 53,88,389,445,636,135 -Protocol TCP;
New-NetFirewallRule -DisplayName "Trust between domains" -Direction outbound -Profile Any -Action Allow -LocalPort 53,88,389,445,636,135 -Protocol TCP

2.Создаем пользователя с админ правами на домене 1:

$pw=ConvertTo-SecureString –AsPlainText -Force -String Passwd!;
New-ADUser -Name "trust" -Description "adtrust" -Enabled $true -AccountPassword $pw;
Add-ADGroupMember -Identity 'Domain Admins' -Members trust

3.Нужно на обеих доменах произвести аналогичные действия
(pwsh) на DC test1.ua
Set-DnsServerPrimaryZone -Name "test1.ua" -SecureSecondaries TransferAnyServer

(pwsh) на DC test2.ua
Set-DnsServerPrimaryZone -Name "test2.ua" -SecureSecondaries TransferAnyServer


1.Конфигурирование DNS 
1.1.Конфигурируем условную пересылку DNS а обеих доменах (на домене 1 и домене 2 выполняем схожие действия) через Powershell:

DC1
Add-DnsServerConditionalForwarderZone -Name test2.ua -MasterServers 192.168.3.119

DC2
Add-DnsServerConditionalForwarderZone -Name test1.ua -MasterServers 192.168.3.119

