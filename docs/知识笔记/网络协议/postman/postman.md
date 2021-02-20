# postman











登陆：  

 http://22.188.12.73:6080/login?username=admin&password=Bdp@boc01! 

下午04:44:50: Content-Type        application/x-www-form-urlencoded; charset=UTF-8 







下午04:45:48: 添加：   

下午04:45:55: http://22.188.12.73:6080/service/plugins/policies 

下午04:46:11: Content-Type      application/json 





下午04:46:59: {"policyType":"0","name":"ssss","isEnabled":true,"policyPriority":0,"policyLabels":[],"description":"","isAuditEnabled":true,"resources":{"database":{"values":["bdp"],"isRecursive":false,"isExcludes":false},"table":{"values":["mqtest"],"isRecursive":false,"isExcludes":false},"column":{"values":["col2"],"isRecursive":false,"isExcludes":false}},"isDenyAllElse":false,"policyItems":[{"users":["bdp"],"accesses":[{"type":"select","isAllowed":true},{"type":"update","isAllowed":true},{"type":"create","isAllowed":true},{"type":"drop","isAllowed":true},{"type":"alter","isAllowed":true},{"type":"index","isAllowed":true},{"type":"lock","isAllowed":true},{"type":"all","isAllowed":true},{"type":"read","isAllowed":true},{"type":"write","isAllowed":true},{"type":"repladmin","isAllowed":true},{"type":"serviceadmin","isAllowed":true},{"type":"tempudfadmin","isAllowed":true},{"type":"refresh","isAllowed":true}]}],"allowExceptions":[],"denyPolicyItems":[],"denyExceptions":[],"service":"hivedev"} 