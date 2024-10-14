# 사전 준비

## 1. 시스템 요구 사항

### 1.1 최소 시스템 리소스

Ansible Automation Platform(이하 AAP) 설치를 위해 다음과 같은 최소 시스템 요구 사항이 필요합니다.
|$\color{lime}{\texttt{리소스 타입}}$|$\color{lime}{\texttt{최소 크기}}$|
|:---|:---|
|`메모리`|16GiB|
|`CPU`|4-core|
|`디스크 공간`|40GiB|
|`디스크 IOPS`|1500|
<br>

### 1.2 운영체제 요구 사항

* RHEL 9.2 이상
* *root*가 아닌 앤서블 사용자
  - sudo 혹은 앤서블 권한 상승을 가진 사용자
  - *root* 사용자로 작업 시, 다음 메시지가 표시됨
    ```
    "the remote user should be a non root user"
    ```
* SSH 공개 키 인증 필요
  - 로컬 설치의 경우, *ansible_connection:local* 사용 가능
* 시스템 접속 시 해당 사용자로 SSH 로그인 필요
  - 다른 사용자로 로그인 후, su 혹은 sudo로 해당 사용자로 변경하면 설치 시 에러남
    ```
    ‘systemctl --user’ : “Failed to connect to bus: No such file or directory”
    ```


## 2. 설치에 필요한 채널

* rhel-9-for-x86_64-appstream-rpms
* rhel-9-for-x86_64-baseos-rpms 
