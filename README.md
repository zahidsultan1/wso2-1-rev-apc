# wso2-1-rev-apc     

APC Wallets	V1 Refill | APC Wallets	V2 Refill | APC Wallets	V3 Loan Generic
APC Wallets	Balance Check
APC Wallets	APC Posting
Channel APIs V1 Bundle Subscription | Channel APIs V2 Bundle Subscription | Channel APIs V3 Loan Generic Bundle Subscription
EasyPaisa Bundle Subscription | HBL Bundle Subscription
Jazz Red APIs Loan Check | Jazz Red APIs Msisdn Info | Jazz Red APIs Bill Details

#API CURLS:

## APC Posting
curl -XPOST 'http://microservices.jazz.com.pk/creditservices/APCRecharge/v101' -H 'ApplicationName: ABC' -H 'Content-Type: application/json' -d '{     "msisdn": "923080149178",     "transactionId": "124",     "amount": 110,     "ServiceUser": "VEONCCABT",     "ServicePassword": "VmVAbmNDQGJUdDE5OTE=" }'

## APC Balance
curl GET 'http://microservices.jazz.com.pk/apcBalance/check/923080149178' -H 'ApplicationName: ABC'

## APC Wallets V1 Refill
curl -iv -POST "http://microservices.jazz.com.pk/services/APCRecharge.APCRechargeHttpSoap11Endpoint" -H "Content-Type: text/xml" -H "SOAPAction: http://tempuri.org/TopUpPayment" -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/"> 	<soapenv:Header/> 	<soapenv:Body> 		<tem:TopUpPayment> 			<!--Optional:--> 			<tem:input><![CDATA[<?xml version="1.0" encoding="UTF-8"?> <ACTIVITY xmlns=""><TransactionId>T943424523763</TransactionId> <ServiceUser>1LOADWSO2</ServiceUser> <ServicePassword>Mobilink@1</ServicePassword> <MSISDN>03080149178</MSISDN> <Amount>100</Amount><Field1></Field1> <Field2>?</Field2></ACTIVITY>]]></tem:input> 		</tem:TopUpPayment> 	</soapenv:Body> </soapenv:Envelope>'

## APC Wallets V2 Refill
curl -iv -POST "http://microservices.jazz.com.pk/services/APCRechargeV2DBSS.APCRechargeV2DBSSHttpSoap11Endpoint" -H "Content-Type: text/xml" -H "SOAPAction: http://tempuri.org/TopUpPayment" -d '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/">    <soapenv:Header/>    <soapenv:Body>       <tem:TopUpPayment>          <!--Optional:-->          <tem:input><![CDATA[<?xml version="1.0" encoding="UTF-8"?> <ACTIVITY xmlns=""><TransactionId>234543455434</TransactionId> <ServiceUser>MEEZANBANKWSO2</ServiceUser><ServicePassword>Mobilink@2222</ServicePassword> <MSISDN>03080149178</MSISDN> <Amount>100</Amount><Field1>03023090013</Field1><Field2>?</Field2></ACTIVITY>]]></tem:input>       </tem:TopUpPayment>    </soapenv:Body> </soapenv:Envelope>'

## APC Wallets V2 Loan Generic
curl -XPOST 'http://microservices.jazz.com.pk/apcrefill/TopUp/refill' -H 'Content-Type: application/json' -H 'ApplicationName: FINCA' -d '{     "Request": {         "TransactionId":"001122",         "ServiceUser":"FINCAWSO2",         "ServicePassword":"MobilinkFINCA@1008",         "MSISDN":"923080149178",         "Amount":100,         "RetailerNumber":"03022230021"     } }'

## Channel API V1 Bundle Subscription
curl -XPOST 'http://microservices.jazz.com.pk/jazzChannel/bundleSubciption' -H 'Content-Type: application/json' -H 'ApplicationName: MEEZANBANK' -d '{     "msisdn": "923080149178",     "price": "55",     "serviceCode": "DataBundles",     "serviceGroup": "DataBundles",     "transaction_id": "098765" }'

## Channel API V2 Bundle Subscription
curl -XPOST 'http://microservices.jazz.com.pk/jazzChannel/V2/bundleSubciption' -H 'Content-Type: application/json' -H 'ApplicationName: MEEZANBANK' -d '{     "msisdn": "923080149178",     "price": "100",     "serviceCode": "DataBundles",     "serviceGroup": "DataBundles",     "transaction_id": "098765" }'

## Channel API V3 Lona Generic Bundle Subscription
curl -XPOST 'http://microservices.jazz.com.pk/Jazzchannel/Generic/bundle' -H 'Content-Type: application/json' -H 'ApplicationName: FINCA' -d '{     "msisdn": "923080149178",     "price": "55",     "serviceGroup": "DataBundles",     "serviceCode": "DailySocial",     "transaction_id": "1234567890"     }'

## JazzRed Loan View V1
curl -XPOST 'https://microservices.jazz.com.pk/ecare/loan/v1/view' -H 'Content-Type: application/json' -H 'ApplicationName: ABC' -d '{     "ID": "0098783838987",     "MSISDN": "923080149178",     "DAID1": "16",     "DAID2": "17" }'

## JazzRed Bill Details
curl GET 'http://microservices.jazz.com.pk/ecare/subscription/billdetails/v1/923080149178/jazz/postpaid' \ -H 'ApplicationName: ABC'

## JazzRed Msisdn Info
curl GET 'http://microservices.jazz.com.pk/ecare/subscription/info/v1/923080149178' -H 'ApplicationName: ABC'

## BundleSubcription JazzCash Wallet
curl -XPOST 'http://microservices.jazz.com.pk/jazz_api/jazzwallet/v1/bundlesubscription' -H 'ApplicationName: ABL' -H 'Content-Type: application/json' -d '{     "msisdn": "923080149178",     "price": "12",     "serviceCode": "SUPERBUN24HRS",     "transaction_id": "1234567895",     "jcAccount": "923086168618",       "mpin": "2626"     }'
