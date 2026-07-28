https://www.youtube.com/watch?v=7Upc4iCsSh4

D:\antn\Admin\Java-Banking-Project\banking-system


cd D:\antn\Admin\Java-Banking-Project\banking-system\transaction-service
mvnw test -Dtest=TransactionServiceApplicationTests


Get-Content src\main\resources\application.yaml | Out-File -FilePath yaml_check.txt -Encoding utf8


Get-Content src\main\resources\application.yaml 

Get-Content kafka_all.txt     

.\mvnw.cmd clean test 2>&1 | Select-String "Kafka" | Out-File kafka_all.txt           

.\mvnw.cmd clean test 2>&1 | Select-String "KafkaAutoConfiguration" -Context 3,3 | Out-File kafka_report.txt

Get-ChildItem -Path src\test\resources -Recurse -ErrorAction SilentlyContinue             

Select-String -Path src\main\java\com\banking\frauddetectionservice\FraudDetectionServiceApplication.java -Pattern "exclude"

.\mvnw.cmd dependency:tree | Select-String "kafka"         


Select-String -Path pom.xml -Pattern "spring-kafka" -Context 3,3              

Select-String -Path src\main\java\com\banking\frauddetectionservice\service\FraudDetectionService.java -Pattern "KafkaTemplate"

Get-Content src\main\java\com\banking\frauddetectionservice\FraudDetectionServiceApplication.java   

cd ..\fraud-detection-service          

 .\mvnw.cmd clean compile         

Get-Content src\main\java\com\banking\transactionservice\entity\TransactionStatus.java     

for ($i=138; $i -lt 175; $i++) { Write-Host "$($i+1): $($lines[$i])" }     

Select-String -Path src\main\java\com\banking\transactionservice\client\*.java -Pattern "interface|block|Block" 

for ($i=0; $i -lt $lines.Count; $i++) { if ($lines[$i] -match "private.*compensateTransaction|void compensateTransaction") { Write-Host "Found at line $($i+1)"; for ($j=$i; $j -lt $i+20; $j++) { Write-Host "$($j+1): $($lines[$j])" } } }     



$lines = Get-Content src\main\java\com\banking\transactionservice\service\TransactionService.java      

Select-String -Path src\main\java\com\banking\transactionservice\service\TransactionService.java -Pattern "compensateTransaction" -Context 0,0    

Get-Content src\main\java\com\banking\transactionservice\service\TransactionService.java | ForEach-Object {$i=0} {$i++; "$i`: $_"} | Select-String -Pattern ":11[5-9]|:12[0-9]"  

for ($i=114; $i -le 129; $i++) { Write-Host "$($i+1): $($lines[$i])" }         .


Get-Content src\main\java\com\banking\transactionservice\service\TransactionService.java | Select-Object -Index (118..129)     

Get-Content src\main\java\com\banking\accountservice\repository\AccountRepository.java | Select-Object -First 12     

mvn clean compile          

Get-Content pom.xml | Select-Object -Skip 94 -First 50    

Select-String -Path pom.xml -Pattern "spring-boot-starter-webmvc-test" -Context 15,2    

Select-String -Path pom.xml -Pattern "spring-boot-starter-webmvc" -Context 2,2       

cat pom.xml | Select-String "spring-boot-starter-webmvc"         

Select-String -Path src\main\java\com\banking\frauddetectionservice\FraudDetectionServiceApplication.java -Pattern "exclude"         

Get-ChildItem -Path src\test\resources -Recurse -ErrorAction SilentlyContinue


.\mvnw.cmd clean install                          

--------------------------------------------------------------------------                                                                                                                                                                                                                                                                                                                                                                   
.\mvnw.cmd -version

.\mvnw.cmd validate

mvn dependency:tree | grep jackson

./gradlew dependencies | grep jackson


docker compose -f docker-compose-kafka.yml up -d

docker compose -f docker-compose-kafka.yml down -v

--------------------------------------------------------------------------

2. Build
Compile
.\mvnw.cmd compile

hoặc

mvn compile
Clean
.\mvnw.cmd clean
Package
.\mvnw.cmd package
Install
.\mvnw.cmd install
Clean Install
.\mvnw.cmd clean install
Skip Test
.\mvnw.cmd clean install -DskipTests
Build một module
.\mvnw.cmd clean install -pl account-service
.\mvnw.cmd clean install -pl transaction-service
3. Test
Chạy toàn bộ test
.\mvnw.cmd test
Chạy một test class
.\mvnw.cmd test -Dtest=TransactionServiceApplicationTests
Chạy nhiều test
.\mvnw.cmd test -Dtest=AccountServiceTests,TransactionServiceTests
Chạy một method
.\mvnw.cmd test -Dtest=TransactionServiceTests#shouldTransferMoney
4. Dependency
Xem dependency tree
.\mvnw.cmd dependency:tree
Lọc Kafka
.\mvnw.cmd dependency:tree | Select-String "kafka"
Lọc Lombok
.\mvnw.cmd dependency:tree | Select-String "lombok"
Lọc Spring Boot
.\mvnw.cmd dependency:tree | Select-String "spring-boot"
Lọc Jackson
.\mvnw.cmd dependency:tree | Select-String "jackson"
Lọc Hibernate
.\mvnw.cmd dependency:tree | Select-String "hibernate"
5. Kiểm tra pom.xml
Tìm dependency
Select-String -Path pom.xml -Pattern "spring-kafka"
Select-String -Path pom.xml -Pattern "lombok"
Select-String -Path pom.xml -Pattern "spring-boot-starter-test"
Select-String -Path pom.xml -Pattern "spring-boot-starter-data-jpa"
Xem context
Select-String -Path pom.xml -Pattern "spring-kafka" -Context 5,5
Parent
Select-String pom.xml -Pattern "<parent>"
DependencyManagement
Select-String pom.xml -Pattern "dependencyManagement"
Plugin
Select-String pom.xml -Pattern "spring-boot-maven-plugin"
6. Đọc file
Xem toàn bộ
Get-Content pom.xml
Xem 20 dòng đầu
Get-Content pom.xml | Select-Object -First 20
Xem 20 dòng cuối
Get-Content pom.xml | Select-Object -Last 20
Xem từ dòng 80
Get-Content pom.xml | Select-Object -Skip 80 -First 30
Đánh số dòng
Get-Content pom.xml | ForEach-Object {$i=0} {$i++; "$i`: $_"}
7. Search trong source code
Tìm KafkaTemplate
Select-String -Path src\main\java\**\*.java -Pattern "KafkaTemplate"
Tìm @KafkaListener
Select-String -Path src\main\java\**\*.java -Pattern "KafkaListener"
Tìm @Service
Select-String -Path src\main\java\**\*.java -Pattern "@Service"
Tìm @Transactional
Select-String -Path src\main\java\**\*.java -Pattern "@Transactional"
Tìm TODO
Select-String -Path src\main\java\**\*.java -Pattern "TODO"
Tìm compensate
Select-String -Path src\main\java\**\*.java -Pattern "compensate"
8. Kiểm tra application.yml
Get-Content src\main\resources\application.yml
Get-Content src\main\resources\application.yaml
Xuất ra file
Get-Content src\main\resources\application.yaml |
Out-File application_dump.txt
9. Xuất log
Build log
.\mvnw.cmd clean install *> build.log
Chỉ Kafka
.\mvnw.cmd clean install 2>&1 |
Select-String "Kafka"
Chỉ ERROR
.\mvnw.cmd clean install 2>&1 |
Select-String "ERROR"
Chỉ Exception
.\mvnw.cmd clean install 2>&1 |
Select-String "Exception"
Chỉ Failed
.\mvnw.cmd clean install 2>&1 |
Select-String "FAILED"
10. Kiểm tra source
Liệt kê Java
Get-ChildItem src\main\java -Recurse
Chỉ entity
Get-ChildItem src\main\java -Recurse |
Where-Object {$_.Name -like "*Entity*"}
Chỉ repository
Get-ChildItem src\main\java -Recurse |
Where-Object {$_.Name -like "*Repository*"}
11. Maven Help
Effective POM
.\mvnw.cmd help:effective-pom
Effective Settings
.\mvnw.cmd help:effective-settings
Active Profiles
.\mvnw.cmd help:active-profiles
12. Debug Maven
Debug mode
.\mvnw.cmd clean install -X
Stacktrace
.\mvnw.cmd clean install -e
Debug + Stacktrace
.\mvnw.cmd clean install -X -e
13. Kiểm tra JDK
java -version
javac -version
mvn -version
echo $env:JAVA_HOME
14. Kiểm tra cache Maven
Get-ChildItem $env:USERPROFILE\.m2\repository
Xóa cache của một thư viện
Remove-Item "$env:USERPROFILE\.m2\repository\org\projectlombok" -Recurse -Force
Remove-Item "$env:USERPROFILE\.m2\repository\org\springframework\kafka" -Recurse -Force

Sau đó:

.\mvnw.cmd clean install
15. Các lệnh rất hữu ích nhưng ít người biết
.\mvnw.cmd dependency:resolve
.\mvnw.cmd dependency:list
.\mvnw.cmd dependency:analyze
.\mvnw.cmd dependency:purge-local-repository
.\mvnw.cmd versions:display-dependency-updates
.\mvnw.cmd help:describe -Dplugin=compiler
.\mvnw.cmd validate
.\mvnw.cmd verify
.\mvnw.cmd clean verify


-------------------------------------------------------------------------------------------

PS D:\antn\Admin\Java-Banking-Project\banking-system> docker ps -a
CONTAINER ID   IMAGE                      COMMAND                  CREATED       STATUS                     PORTS                                                                                                         NAMES
40db39fe8ba5   main-backend               "python main.py"         13 days ago   Exited (1) 13 days ago                                                                                                                   main-backend-1
77e6e2a59067   main-queue                 "python consumer.py"     13 days ago   Exited (1) 13 days ago                                                                                                                   main-queue-1
9c753e863880   admin-queue                "python consumer.py"     13 days ago   Exited (1) 13 days ago                                                                                                                   admin-queue-1
229bcfef1085   admin-backend              "python manage.py ru…"   13 days ago   Exited (255) 13 days ago   0.0.0.0:8000->8000/tcp                                                                                        admin-backend-1
6752a9d0a78b   mysql:5.7.22               "docker-entrypoint.s…"   13 days ago   Up 33 minutes              0.0.0.0:33066->3306/tcp, [::]:33066->3306/tcp                                                                 admin-db-1
66ff04369a13   mysql:5.7.22               "docker-entrypoint.s…"   13 days ago   Up 33 minutes              0.0.0.0:33067->3306/tcp, [::]:33067->3306/tcp                                                                 main-db-1
fcae79bcd786   rabbitmq:3-management      "docker-entrypoint.s…"   2 weeks ago   Exited (255) 6 days ago    4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, 15671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp   rabbitmq
c46517c20703   postgres                   "docker-entrypoint.s…"   2 weeks ago   Up 33 minutes              0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp                                                                   postgres_container
4a9cc482bc51   openzipkin/zipkin          "start-zipkin"           2 weeks ago   Exited (255) 2 weeks ago   9410/tcp, 0.0.0.0:9411->9411/tcp                                                                              zipkin
79b542543774   openzipkin/zipkin          "start-zipkin"           2 weeks ago   Exited (143) 2 weeks ago                                                                                                                 naughty_saha
08c38ad64ff3   dpage/pgadmin4             "/entrypoint.sh"         2 weeks ago   Up 33 minutes              0.0.0.0:5050->80/tcp, [::]:5050->80/tcp                                                                       pgadmin_container
f8d96e77c55f   dpage/pgadmin4             "/entrypoint.sh"         3 weeks ago   Exited (255) 2 weeks ago   80/tcp, 443/tcp                                                                                               pgadmin
2e0043dc894f   postgres                   "docker-entrypoint.s…"   3 weeks ago   Exited (255) 2 weeks ago   5432/tcp                                                                                                      db
9fef7a5ab16e   ditimgiadinh/jobappimage   "/cnb/process/web"       3 weeks ago   Exited (130) 3 weeks ago                                                                                                                 hungry_chebyshev
PS D:\antn\Admin\Java-Banking-Project\banking-system> docker compose down -v
empty compose file

What's next:
    Debug this Compose error with Gordon → docker ai "help me fix this compose error"
PS D:\antn\Admin\Java-Banking-Project\banking-system> docker compose up -d
time="2026-07-27T15:30:42-03:00" level=warning msg="D:\\antn\\Admin\\Java-Banking-Project\\banking-system\\docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion"
[+] up 32/35
 ✔ Image confluentinc/cp-kafka:7.4.0      Pulled                                                                                                                                                          1036.7s
 ✔ Image mysql:8.0                        Pulled                                                                                                                                                           850.1s
 ✔ Image confluentinc/cp-zookeeper:7.4.0  Pulled                                                                                                                                                          1043.6s
 ✔ Network banking-system_banking-network Created                                                                                                                                                            1.0s
 ✔ Volume banking-system_mysql-data       Created                                                                                                                                                            0.2s
 ✔ Container redis                        Started                                                                                                                                                           12.4s
 - Container mysql                        Starting                                                                                                                                                          12.4s
 ✔ Container zookeeper                    Started                                                                                                                                                           12.4s
 ✔ Container kafka                        Created                                                                                                                                                            0.4s
Error response from daemon: ports are not available: exposing port TCP 0.0.0.0:3306 -> 127.0.0.1:0: listen tcp 0.0.0.0:3306: bind: Only one usage of each socket address (protocol/network address/port) is normally permitted.

What's next:
    Debug this Compose error with Gordon → docker ai "help me fix this compose error"
PS D:\antn\Admin\Java-Banking-Project\banking-system> docker compose up -d
time="2026-07-27T15:50:47-03:00" level=warning msg="D:\\antn\\Admin\\Java-Banking-Project\\banking-system\\docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion"
[+] up 4/4
 ✔ Container zookeeper Running                                                                                                                                                                               0.0s
 ✔ Container redis     Running                                                                                                                                                                               0.0s
 ✔ Container mysql     Started                                                                                                                                                                               0.8s
 ✔ Container kafka     Started                                                                                                                                                                               0.8s
PS D:\antn\Admin\Java-Banking-Project\banking-system>

-------------------------------------------------------------------------------------------------------------------

                                                                                                                               