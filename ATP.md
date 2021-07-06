# Queries
* https://github.com/microsoft/Microsoft-threat-protection-Hunting-Queries
* https://github.com/jangeisbauer/AdvancedHunting

Notebook converter from sigma rule to kql query.
* https://techcommunity.microsoft.com/t5/azure-sentinel/importing-sigma-rules-to-azure-sentinel/ba-p/657097#


## Identity
### Usuarios desde varias ubicaciones
IdentityInfo 
| summarize cities=dcount(City), ciudades=make_set(City) by AccountUpn
| order by cities 

### Ubicaciones con varios usuarios
IdentityInfo 
| summarize users=dcount(AccountUpn) by City
| order by users asc 

## Email
### Adjuntos con extensiones
EmailAttachmentInfo 
| where FileName contains ".xlsm" //FileName contains ".exe" or FileName contains ".ps1" or FileName contains ".bat" //or FileName contains ".zip"
| where FileName !contains ".xlsx"
| where SenderFromAddress !contains "<company>"
