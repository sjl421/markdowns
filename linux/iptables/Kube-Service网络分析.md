# Kube-Service网络分析

//第一次dnt,将访问adbops-service clusterIp:port的方法 dnat到下一条链上	
-A KUBE-SERVICES -d 10.100.92.62/32 -p tcp -m comment --comment "default/adbops-service:http cluster IP" -m tcp --dport 8080 -j KUBE-SVC-TXDRYUK3PIVY2MYX	

//继续跳转
-A KUBE-SVC-TXDRYUK3PIVY2MYX -m comment --comment "default/adbops-service:http" -j KUBE-SEP-VGX56CCCCWZBBO2O
//添加标识
-A KUBE-SEP-VGX56CCCCWZBBO2O -s 10.244.10.2/32 -m comment --comment "default/adbops-service:http" -j KUBE-MARK-MASQ
//跳转
-A KUBE-SEP-VGX56CCCCWZBBO2O -p tcp -m comment --comment "default/adbops-service:http" -m tcp -j DNAT --to-destination 10.244.10.2:8080