# AWS 환경에 k8s CI/CD 구축하기

![**CI**](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bd890a9d-b21b-4bbe-9da0-1177a06d4ea9/Untitled.png)

**CI**

![**CD**](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd748ce0-6337-4e7f-b275-e2463b575db7/Untitled.png)

**CD**

## TO-DO

- [x]  [EC2 생성 및 ElasticIP 설정](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [k8s 환경 구성](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [Jenkins 컨테이너 실행 및 빌드 환경 세팅](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [Argo CD컨테이너 실행 및 Item 설정](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [gitea 컨테이너 실행 및 repo 설정](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [gitea repo 생성](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [도커 레포지토리 생성 및 Dockerfile 작성](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [Jenkins Pipeline 작성](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [Jenkins Pipeline 테스트](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [WebHook 설정](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)

---

## 궁금한

- [x]  Jenkins Pipeline 으로 애플리케이션 repo git clone, argocd yaml repo git clone, 다른 경로에 하나?
    
    → 프로젝트 다운 및 빌드 해야하므로 별도의 공간에서 작업 필요
    
    ```groovy
     dir('app-repo') {}
     dir('yarm-repo') {}
    ```
    
    ```bash
    root@ea4b8a7c6ad7:/var/jenkins_home/workspace/tomcat-project# ls -al | grep repo
    \drwxr-xr-x. 11 root    root      4096 May 15 12:21 app-repo
    drwxr-xr-x.  2 root    root         6 May 15 12:22 app-repo@tmp
    drwxr-xr-x.  3 root    root        87 May 15 12:22 yarml-repo
    drwxr-xr-x.  2 root    root         6 May 15 12:22 yarml-repo@tmp
    ```
    
- [x]  그 전에 빌드한 거는 다시 빌드할 때 밀리나? 아님 빌드 완료한 후에 미나?
    
    → Jenkins의 기본 동작 방식은 빌드 시 이전 빌드 결과를 삭제, “Discard Old Builds” 설정에 의해 제어, 설정을 수정하여 이전 빌드를 유지하도록 변경할 수도 있음
    
- [ ]  컨테이너 Jenkins 에서 Docker를 사용하려면 Jenkins 컨테이너 실행 시 Host OS 의 Docker를 사용할 수 있도록 별도의 설정이 필요함?
    
    → 컨테이너에서 Host Docker 에 영향을 주면 안 됨, Host Docker 를 공유..?
    
    [Docker](https://www.jenkins.io/doc/book/installing/docker/#downloading-and-running-jenkins-in-docker)
    
- [x]  argo yaml 에 버전을 뭐라고 적나
- [x]  여러번 배포 시 ArgoCD 에 deployment 가 여러개 뜨는데 문제 없는가, 삭제 처리 해줘야하나
    - argo cd 에서 이전의 replicaSet이 남아 있음
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f266690f-0821-4381-a942-f087a0cc6f57/Untitled.png)
        
        ```bash
        # k8s cluster 에서도 rs가 남아 있음을 확인..
        k get rs -n argocd-deploy 
        NAME                           DESIRED   CURRENT   READY   AGE
        tomcat-deployment-5b8576c6cd   0         0         0       2d19h
        tomcat-deployment-5c66b85cb    0         0         0       3d16h
        tomcat-deployment-74c4db94c    0         0         0       3d1h
        tomcat-deployment-76bc994475   0         0         0       3d2h
        tomcat-deployment-7b54fb7dbf   2         2         2       10m
        tomcat-deployment-7cfc89dd77   0         0         0       14m
        tomcat-deployment-854dcfc696   0         0         0       3d2h
        tomcat-deployment-85987c7946   0         0         0       3d2h
        tomcat-deployment-8679c89c8f   0         0         0       2d20h
        tomcat-deployment-8b5db5bcf    0         0         0       3d2h
        tomcat-deployment-ccd4bf6d9    0         0         0       3d2h
        ```
        
    
    → deployment default `revisionHistoryLimit` 이 10이라 그럼, 조정 가능
    
    ```bash
    revisionHistoryLimit: 10
    ```
    
- [x]  서버에서 sudo 하고 jenkins container를 올릴때와 sudo -i 상태인 root 계정으로 jenkins container 를 올릴때 차이
    - sudo, sudo-i 비교
        
        
        |  | sudo | sudo -i |
        | --- | --- | --- |
        | root 권한 | 임시 root 권한 | root 권한 |
        | 환경 변수 | 현재 사용자의 환경변수 상속 | root 계정의 환경변수 사용 |
        | 홈 디렉토리 | 현재 사용자의 홈 디렉토리가 Jenkins 컨테이너의 홈 디렉토리로 사용 | root 계정의 홈 디렉토리가 Jenkins 컨테이너의 홈 디렉토리로 사용됨 |
- [ ]  쿠버네티스 설치 시 swapoff 하는 이유
    - 성능 향상, 메모리 사용의 예측 가능성, OOM Killer 문제 예방 등
        1. 성능: 스왑은 운영 체제의 가상 메모리 기능으로, 메모리 부족 상황에서 물리적인 디스크 공간을 사용하여 메모리를 보조하는 역할을 합니다. 그러나 스왑을 사용하면 디스크 I/O가 발생하고 응답 시간이 늘어날 수 있으며, 컨테이너화된 애플리케이션의 성능을 저하시킬 수 있습니다. 쿠버네티스는 높은 성능과 신뢰성을 위해 메모리 사용을 최적화하기 때문에, 스왑을 비활성화하는 것이 권장됩니다.
        2. 예측 가능한 메모리 사용: 쿠버네티스 클러스터는 노드 간에 작업을 분산시키고 메모리 요구 사항을 관리합니다. 메모리 요구 사항을 예측하기 어려운 스왑은 쿠버네티스 클러스터의 리소스 관리를 복잡하게 만들 수 있습니다. 따라서 스왑을 비활성화하여 메모리 사용을 예측 가능하고 일관되게 유지하는 것이 중요합니다.
        3. OOM Killer 문제 예방: 스왑을 사용하면 메모리 부족 상황에서 운영 체제가 Out-of-Memory (OOM) Killer라고 불리는 프로세스를 사용하여 메모리를 확보하려고 할 수 있습니다. 이는 가장 많은 메모리를 사용하는 프로세스를 강제로 종료시키는 것입니다. 그러나 스왑이 비활성화되어 있으면 OOM Killer가 동작하지 않으므로, 예기치 않은 애플리케이션 종료를 방지할 수 있습니다.

---

## 개선하거나 추가하고 싶은

- [x]  argo yaml 에 버전을 뭐라고 적나
    - v${env.BUILD_ID} 로 태그하여 도커 이미지 업로드하고, argo yaml 파일의 버전태그를 패턴으로 찾아서 현재 버전과 비교하여 다를 경우 현재 버전으로 소스 업데이트 커밋
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fb0a2dc5-af68-4fb6-abd3-a498a5c13ade/Untitled.png)
        
- [x]  jenkins pipeline script 에 계정 정보 노출되지 않도록(프로젝트에  업로드하여 버전관리 하는것을 고려해서)
    - credential 에 등록 후 사용
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/28827bd4-17ad-474f-9eca-96b3f1c80a3c/Untitled.png)
        
        - gitea 는 personal token 발급 후 Jenkins credential 에 등록 후 사용
        
        ```groovy
        withCredentials([usernamePassword(credentialsId: 'gitea_token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        	sh 'git push http://${USERNAME}:${PASSWORD}@13.124.203.239:3000/montoring/argocd-yaml.git'
        }
        ```
        
        - docker 는 계정정보를 Jenkins credential 에 등록 후 사용
        
        ```groovy
        withCredentials([usernamePassword(credentialsId: 'docker_login_info', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        	sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} ."
        	sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
        	sh "docker push ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG}"
        }
        ```
        
        - 이메일 등 개인정보는 환경변수에 등록하거나 credential 에 등록 후 사용
        
        ```groovy
        withCredentials([usernamePassword(credentialsId: 'git_email', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        sh 'git config user.email ${USERNAME}'
        }
        withCredentials([usernamePassword(credentialsId: 'git_name', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        	sh 'git config user.name ${USERNAME}'
        }
        ```
        
- [x]  jenkinsfile 프로젝트에 포함하여 버전관리
    - 프로젝트에 Jenkinsfile을 생성하지 않고 build 시 에러 발생
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f0b8991-bdcb-4e75-a007-360eb178b4e2/Untitled.png)
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/96c8ee4d-122c-4fa0-b2d9-28ae8d1245d5/Untitled.png)
        
    - 프로젝트에 Jenkinsfile을 생성하고 build 시 성공
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/52c02d17-6e61-4dae-824f-bc4c6e7cd42e/Untitled.png)
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c9e7f46-1b97-43f6-be19-55205ea60800/Untitled.png)
        
- [x]  jenkins 빌드 결과 slack 으로 알림 전송
    - jenkins, slack 연동
        - slack jenkins CI 앱 추가
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd783811-4637-40b4-ba58-5779280c767c/Untitled.png)
        
        - Jenkins 에  slack Jenkin CI token credential 등록(Secret text type으로)
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2123ac93-0fd4-4dd6-a79e-95bec4700faa/Untitled.png)
            
        - Jenkins `Slack Notification Plugin` 설치
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f7db47e4-a5de-4419-b9bc-e1c8bb2285a0/Untitled.png)
            
        - Jenkins Slack 시스템 설정
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/47654535-b55a-41ec-995b-3a8b9bbc693d/Untitled.png)
            
        - Jenkinsfile 에 slack 전송 post 구문 추가
            
            ```groovy
            post {
            	failure {
            		slackSend channel: '#k8s-cicd',
            		color: 'danger',
            		message: "${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} Build failed!",
            		tokenCredentialId: 'slack_webhook'
            	}
            
            	success {
            		slackSend channel: '#k8s-cicd',
            		color: 'good',
            		message: "${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} Build successful!",
            		tokenCredentialId: 'slack_webhook'
            	}
            }
            ```
            
        - build 및 slack 메세지 확인
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bb6f2bea-9866-4e13-aef5-9ebc80327c7d/Untitled.png)
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/918cd533-39e8-45b8-a6c7-447c8b48ca27/Untitled.png)
            
- [x]  jenkins pipeline 스크립트 리팩토링
    - domain, repo, docker image 등 변수화
        
        ```python
        environment {
                GITEA_DOMAIN = "13.124.203.239:3000/montoring/"
                APP_REPO = "tomcat-project.git"
                YAML_REPO = "argocd-yaml.git"
                DOCKER_IMAGE = "hyewone/mentoring_project"
                DOCKER_IMAGE_TAG = "v${env.BUILD_ID}"
            }
        ```
        
- [ ]  그라파나 프로메테우스 모니터링
- [ ]  Deployment pod autoscaling
- [ ]  서버, 컨테이너, k8s 백업
    - [argocd는 PV가 불필요](https://github.com/argoproj/argo-cd/issues/4255)
    - hostpath 를 파드에 매핑해두면 컨테이너가 재실행되도 이전 설정 유지
        
        ```bash
        k create namespace jenkins
        namespace/jenkins created
        
        k create deployment jenkins-server --namespace jenkins --image=jenkins/jenkins --dry-run=client -o yaml > jenkins-server.yaml
        ls
        README.md  jenkins-server.yaml  tomcat-project
        
        vim jenkins-server.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          creationTimestamp: null
          labels:
            app: jenkins-server
          name: jenkins-server
          namespace: jenkins
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: jenkins-server
          strategy: {}
          template:
            metadata:
              creationTimestamp: null
              labels:
                app: jenkins-server
            spec:
              containers:
              - image: jenkins/jenkins
                name: jenkins
                resources: {}
                volumeMounts:
                - name: jenkins-home
                  mountPath: /var/jenkins_home
              volumes:
              - name: jenkins-home
                hostPath:
                  path: /home/centos/var/jenkins_home
        status: {}
        
        k apply -f jenkins-server.yaml 
        deployment.apps/jenkins-server created
        
        k get pod -n jenkins 
        NAME                             READY   STATUS   RESTARTS       AGE
        jenkins-server-68b989c97-84zdl   0/1     Error    5 (100s ago)   3m26s
        
        k logs jenkins-server-68b989c97-84zdl -n jenkins 
        touch: cannot touch '/var/jenkins_home/copy_reference_file.log': Permission denied
        Can not write to /var/jenkins_home/copy_reference_file.log. Wrong volume permissions?
        
        ls -al
        drwxr-x--x. 3 centos centos    26  5월 17 09:35 var
        
        sudo chmod 757 var
        
        ls -al
        drwxr-xrwx. 3 centos centos    26  5월 17 09:35 var
        
        k get pod -n jenkins 
        NAME                             READY   STATUS    RESTARTS   AGE
        jenkins-server-99c9dcb8f-f2ckr   1/1     Running   0          3m44s
        
        vim jenkins-svc.yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: jenkins-svc
          namespace: jenkins
        spec:
          type: NodePort
          selector:
            app: jenkins-server
          ports:
          - port: 8080
            targetPort: 8080
            nodePort: 30000
        
        k get svc -n jenkins jenkins-svc 
        NAME          TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        jenkins-svc   NodePort   10.96.84.123   <none>        8080:30000/TCP   11m
        ```
        
        ```bash
        curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
        chmod +x get_helm.sh
        ./get_helm.sh
        
        [WARNING] Could not find git. It is required for plugin installation.
        Downloading https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz
        Verifying checksum... Done.
        Preparing to install helm into /usr/local/bin
        helm installed into /usr/local/bin/helm
        helm not found. Is /usr/local/bin on your $PATH?
        Failed to install helm
        
        vi ~/.bashrc
        [root@master ~]# export PATH=/usr/local/bin:$PATH
        [root@master ~]# vi ~/.bashrc
        [root@master ~]# source ~/.bashrc
        [root@master ~]# helm version
        version.BuildInfo{Version:"v3.12.0", GitCommit:"c9f554d75773799f72ceef38c51210f1842a1dea", GitTreeState:"clean", GoVersion:"go1.20.3"}
        
        helm repo add gitea-charts https://dl.gitea.io/charts/
        "gitea-charts" already exists with the same configuration, skipping
        [centos@master var]$ helm pull gitea-charts/gitea
        
        tar -xzvf gitea-8.3.0.tgz
        
        vim values.yaml
        
        persistence:
          enabled: false
          hostPath:
            path: /home/centos/var/gitea
        
        k create namespace gitea
        namespace/gitea created
        
        helm install gitea . -n gitea
        coalesce.go:175: warning: skipped value for memcached.initContainers: Not a table.
        NAME: gitea
        LAST DEPLOYED: Wed May 17 11:22:10 2023
        NAMESPACE: gitea
        STATUS: deployed
        REVISION: 1
        NOTES:
        1. Get the application URL by running these commands:
          export NODE_PORT=$(kubectl get --namespace gitea -o jsonpath="{.spec.ports[0].nodePort}" services gitea)
          export NODE_IP=$(kubectl get nodes --namespace gitea -o jsonpath="{.items[0].status.addresses[0].address}")
          echo http://$NODE_IP:$NODE_PORT
        
        k get pod -n gitea
        NAME                              READY   STATUS    RESTARTS   AGE
        gitea-0                           0/1     Pending   0          88s
        gitea-memcached-ff5b989d9-vg9tl   1/1     Running   0          88s
        gitea-postgresql-0                0/1     Pending   0          88s
        
        k describe pod gitea-0 -n gitea
        Warning  FailedScheduling  30s   default-scheduler  0/1 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod..
        
        # 아무리 그냥 hostpath로 해보려해도 helm이 gitea 권고사항인 pvc 로 생성함
        
        helm install gitea-server \
        --set adminUsername=admin,giteaPassword=admin,postgresql.auth.rootPassword=admin, \
        --set persistence.existingClaim=gitea-pvc \ 
        --set persistence.existingPostgresClaim=gitea-postgres-pvc \ 
        --set service.type=NodePort,service.nodePorts.http=30001 \
        oci://registry-1.docker.io/bitnamicharts/gitea -n gitea
        
        helm install gitea-server \
        --set adminUsername=admin \
        --set giteaPassword=admin \
        --set postgresql.auth.rootPassword=admin \
        --set persistence.existingClaim=gitea-pvc \
        --set persistence.existingPostgresClaim=gitea-postgres-pvc \
        --set service.type=NodePort \
        --set service.nodePorts.http=30001 \
        oci://registry-1.docker.io/bitnamicharts/gitea -n gitea
        
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: gitea-pvc
          namespace: gitea
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 10Gi
        ---
        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: gitea-pv
          namespace: gitea
        spec:
          capacity:
            storage: 10Gi
          accessModes:
            - ReadWriteOnce
          persistentVolumeReclaimPolicy: Retain
          hostPath:
            path: "/home/centos/var/gitea"
        
        k apply -f pv-pvc-gitea.yaml
        
        k describe pvc gitea-pvc -n gitea 
        Name:          gitea-pvc
        Namespace:     gitea
        StorageClass:  
        Status:        Pending
        Volume:        
        Labels:        <none>
        Annotations:   <none>
        Finalizers:    [kubernetes.io/pvc-protection]
        Capacity:      
        Access Modes:  
        VolumeMode:    Filesystem
        Used By:       gitea-server-76dd549787-nsnsz
        Events:
          Type    Reason         Age                  From                         Message
          ----    ------         ----                 ----                         -------
          Normal  FailedBinding  6s (x23 over 5m23s)  persistentvolume-controller  no persistent volumes available for this claim and no storage class is set
        
        ```
        
        ```bash
        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: gitea-pv
        spec:
          capacity:
            storage: 5Gi
          accessModes:
            - ReadWriteOnce
          hostPath:
            path: "/home/centos/var/gitea"
        ---
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: gitea-pvc
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 5Gi
          storageClassName: ""
          volumeName: gitea-pv
        ---
        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: gitea-postgres-pv
        spec:
          capacity:
            storage: 5Gi
          accessModes:
            - ReadWriteOnce
          hostPath:
            path: "/home/centos/var/gitea-postgres"
        ---
        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: gitea-postgres-pvc
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 5Gi
          storageClassName: ""
          volumeName: gitea-postgres-pv
        ```
        
- [ ]  Deployment “revisionHistoryLimit” 조정

---

## 이런 문제 있었어요

- gitea 인증
    
    → 실제로 jenkins 서버에서 gitea로 push 시도하니 계정 정보를 필요로함
    
    ```bash
    /var/jenkins_home/workspace/tomcat-project/yarml-repo# git push -u origin main
    Username for '[http://13.124.203.239:3000](http://13.124.203.239:3000/)':
    ```
    
    - gitea personal token 발급
        - 필요한 권한을 필히 주어야 함
    - jenkins 에 credential 등록
    - pipeline 수정
    
    ```bash
    withCredentials([usernamePassword(credentialsId: 'gitea_token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            sh 'git push http://${USERNAME}:${PASSWORD}@13.124.203.239:3000/montoring/argocd-yaml.git'
                        }
    ```
    
- Jenkins 컨테이너에서 Docker 일반적 방식으로 설치 안 됨
- Webhook 전달 시험 실패
    - gitea 에서 jenkins 로 webhook 요청 시 jenkins 프로젝트에 설정된 token과 동일한 값을 보내야함
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6d34bf93-9655-4224-844a-e1314ef50b52/Untitled.png)
    

---

- hostname “master” could not be reached
    
    ```bash
    [WARNING Hostname]: hostname "master" could not be reached
    [WARNING Hostname]: hostname "master": lookup master on 172.31.0.2:53: no such host
    ```
    
    ```bash
    cat /etc/hostname
    hostname set-hostname master
    cat <<EOF >> /etc/hosts
    127.0.0.1   localhost
    172.31.54.3 master
    EOF
    ```
    
- ERROR CRI
    
    ```bash
    [ERROR CRI]: container runtime is not running: output: time="2023-05-10T14:02:50Z" level=fatal msg="validate service connection: CRI v1 runtime API is not implemented for endpoint \"unix:///var/run/containerd/containerd.sock\": rpc error: code = Unimplemented desc = unknown service runtime.v1.RuntimeService"
    ```
    
    ```bash
    # Containerd configuration for Kubernetes
    cat <<EOF | sudo tee -a /etc/containerd/config.toml
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
    EOF
    
    sudo sed -i 's/^disabled_plugins \=/\#disabled_plugins \=/g' /etc/containerd/config.toml
    sudo systemctl restart containerd
    
    # 소켓이 있는지 확인한다.
    ls /var/run/containerd/containerd.sock
    ```
    
- NotReady Node
    
    ```bash
    sudo kubeadm init
    
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    kubectl get node
    NAME     STATUS     ROLES           AGE     VERSION
    master   NotReady   control-plane   2m55s   v1.27.1
    
    k describe node master
    container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized
    
    sudo kubeadm reset
    
    sudo rm -rf /etc/cni/net.d
    sudo rm -rf /opt/cni/bin/calico calico-ipam
    
    # CNI 설치 필요
    # calico CNI install https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16
    
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/tigera-operator.yaml
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/custom-resources.yaml
    
    # calico pod 가 모두 running 상태가 되어야함
    watch kubectl get pods -n calico-system                                                                                                                                             Wed May 10 16:05:58 2023
    
    NAME                                       READY   STATUS    RESTARTS   AGE
    calico-kube-controllers-789dc4c76b-wg8vf   1/1     Running   0          2m46s
    calico-node-mrps9                          1/1     Running   0          2m47s
    calico-typha-67d5697c97-ntrwg              1/1     Running   0          2m47s
    csi-node-driver-g5pj5                      2/2     Running   0          2m46s
    
    k get node
    NAME     STATUS   ROLES           AGE   VERSION
    master   Ready    control-plane   11m   v1.27.1
    ```
    

---

- EC2 out of memory
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8054765c-8a45-46f1-bac8-9eb5ff372226/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/56e27aa3-6192-480e-9d9c-610a00222b9c/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/466d9daa-f4aa-42b3-898c-cc03d89969c1/Untitled.png)
    

---

## 멘토님 피드백

<aside>
💡 Jenkins Pipeline 을 SCM 방식으로 구성하여 버전관리, CI, CD 한 점을 어필하면 좋을 것 같다.
환경은 다 다르기에 내가 그 환경에 맞춰 경험할 수 없다. 난 이렇게 해봤다를 잘 어필하는 것이 중요

</aside>

- [ ]  k8s 환경에서 Docker 로 컨테이너를 띄우는 것은 일반적이지 않음, 현재 도커로 띄운 Jenkins, gitea 컨테이너를 k8s Pod 로 구성해보기
- [ ]  k8s 자원의 matrics 모니터링이 가능하게 설정하고, HPA 구성 및 테스트해보기
- [ ]  비용이 걱정된다면, 인스턴스는 중지/pod는/컨테이너는
- [ ]  AWS VPC 환경도 구성해보기

---

- 모듈 공부 책보다는 공홈을 추천
- AWS 환경 퍼블릭 클라우드면 EKS 사용
- EKS 는 마스터노드 관리를 안 해줘도 됨, EKS 마스터노드에서 kube-system static pod가 뜨지 않음
- 쿠버네티스 설치 shell script 파일로 만들어서 함, 시간 지나면 변경사항에 의해 수정해야 하기도함
- 랜처
- Terraform은 EC2, EKS, VPC, subnet 등 구성 시 사용

---

## DONE

### EC2 생성 및 ElasticIP 설정

- EC2 생성
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/63a30c05-7264-40ce-845f-3e9865ff5ac7/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1603bb9d-6f7c-4b6f-8bea-c082e55aed54/Untitled.png)
    
- ElasticIP 생성 및 연결
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2cd57a16-0d68-47fa-b256-1db92c181ac2/Untitled.png)
    
- 인스턴스 ssh 접속
    
    ```bash
    ssh -i hw_jenkins.pem centos@13.124.203.239
    ```
    

---

### k8s 환경 구성

- swap 비활성화
    
    ```bash
    # swap 비활성화
    sudo swapoff -a # 현재 시스템에 적용(리부팅하면 재설정 필요)
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab # 리부팅 필수
    ```
    
- iptables 커널 설정
    
    ```bash
    # iptables 커널 설정
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter
    
    # 필요한 sysctl 파라미터를 설정하면, 재부팅 후에도 값이 유지된다.
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF
    
    # 재부팅하지 않고 sysctl 파라미터 적용하기
    sudo sysctl --system
    ```
    
- hostname 지정 및 hosts 파일 설정
    
    ```bash
    sudo -i
    hostnamectl set-hostname master
    cat <<EOF >> /etc/hosts
    127.0.0.1   localhost
    172.31.54.3 master
    EOF
    ```
    
- containerd 설치
    
    ```bash
    # docker 설치
    sudo rpm --import https://download.docker.com/linux/centos/gpg
    
    sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
    [dockerrepo]
    name=Docker Repository
    baseurl=https://download.docker.com/linux/centos/7/x86_64/stable/
    enabled=1
    gpgcheck=1
    gpgkey=https://download.docker.com/linux/centos/gpg
    EOF
    
    sudo systemctl enable --now containerd
    
    # Containerd configuration for Kubernetes
    cat <<EOF | sudo tee -a /etc/containerd/config.toml
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
    EOF
    
    sudo sed -i 's/^disabled_plugins \=/\#disabled_plugins \=/g' /etc/containerd/config.toml
    sudo systemctl restart containerd
    
    # 소켓이 있는지 확인한다.
    ls /var/run/containerd/containerd.sock
    ```
    
- docker 설치 (안 해도됨)
    
    ```bash
    sudo yum install -y yum-utils
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    
    sudo yum install docker-ce docker-ce-cli docker-buildx-plugin docker-compose-plugin
    
    sudo systemctl enable --now docker
    
    sudo docker login
    Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
    Username: hyewone
    Password: 
    WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credentials-store
    
    Login Succeeded
    ```
    
- kubelet, kubeadm, kubectl 설치
    
    ```bash
    # kubelet, kubeadm, kubectl 설치
    cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
    enabled=1
    gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    exclude=kubelet kubeadm kubectl
    EOF
    
    # permissive 모드로 SELinux 설정(효과적으로 비활성화)
    sudo setenforce 0
    sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    
    sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
    
    sudo systemctl enable --now kubelet
    
    kubeadm version
    kubeadm version: &version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"clean", BuildDate:"2023-04-14T13:20:04Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
    kubectl version
    WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
    Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"clean", BuildDate:"2023-04-14T13:21:19Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
    Kustomize Version: v5.0.1
    
    sudo systemctl status kubelet
    ● kubelet.service - kubelet: The Kubernetes Node Agent
       Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
      Drop-In: /usr/lib/systemd/system/kubelet.service.d
               └─10-kubeadm.conf
       Active: activating (auto-restart) (Result: exit-code) since 수 2023-05-10 12:57:58 UTC; 4s ago
         Docs: https://kubernetes.io/docs/
      Process: 30384 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
     Main PID: 30384 (code=exited, status=1/FAILURE)
    
     5월 10 12:57:58 ip-172-31-54-3.ap-northeast-2.compute.internal systemd[1]: ...
     5월 10 12:57:58 ip-172-31-54-3.ap-northeast-2.compute.internal systemd[1]: ...
    Hint: Some lines were ellipsized, use -l to show in full.
    ```
    
- CNI 설치
    
    ```bash
    # CNI 설치 필요
    # calico CNI install https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16
    
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    kubectl get node
    NAME     STATUS     ROLES           AGE   VERSION
    master   NotReady   control-plane   66s   v1.27.1
    
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/tigera-operator.yaml
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/custom-resources.yaml
    
    # calico pod 가 모두 running 상태가 되어야함
    watch kubectl get pods -n calico-system
    
    NAME                                       READY   STATUS    RESTARTS   AGE
    calico-kube-controllers-789dc4c76b-wg8vf   1/1     Running   0          2m46s
    calico-node-mrps9                          1/1     Running   0          2m47s
    calico-typha-67d5697c97-ntrwg              1/1     Running   0          2m47s
    csi-node-driver-g5pj5                      2/2     Running   0          2m46s
    
    k get node
    NAME     STATUS   ROLES           AGE   VERSION
    master   Ready    control-plane   11m   v1.27.1
    ```
    
- kubectl 자동완성 및 alias 설정
    
    ```bash
    sudo yum install bash-completion
    source /usr/share/bash-completion/bash_completion
    echo 'source <(kubectl completion bash)' >>~/.bashrc
    echo 'alias k=kubectl' >>~/.bashrc
    echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
    ```
    
- Taint 제거
    
    ```bash
    # master node 에 스케줄링 하기 위해
    kubectl taint nodes --all node-role.kubernetes.io/control-plane-
    kubectl taint nodes --all node-role.kubernetes.io/master-
    ```
    

---

### Jenkins 컨테이너 실행 및 빌드 환경 세팅

- Jenkins 컨테이너 실행
    - 호스트 도커 sock 을 Jenkins 컨테이너에서 사용하도록 설정 필요
    - jenkins-blueocean (jenkins 서버) 컨테이너 실행 시 로컬 볼륨을 매핑하는 -v 옵션을 주었기에 컨테이너가 중지 후 restart 되어도 jenkins 관련 이전 설정 남아 있음
    
    ```bash
    # https://www.jenkins.io/doc/book/installing/docker/#downloading-and-running-jenkins-in-docker
    
    docker network create jenkins
    
    docker run \
      --name jenkins-docker \
      --rm \
      --detach \
      --privileged \
      --network jenkins \
      --network-alias docker \
      --env DOCKER_TLS_CERTDIR=/certs \
      --volume jenkins-docker-certs:/certs/client \
      --volume jenkins-data:/var/jenkins_home \
      --publish 2376:2376 \
      docker:dind \
      --storage-driver overlay2
    
    ---dockerfile--------------------------------------------------------------------
    
    FROM jenkins/jenkins:2.387.3
    USER root
    RUN apt-get update && apt-get install -y lsb-release
    RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
      https://download.docker.com/linux/debian/gpg
    RUN echo "deb [arch=$(dpkg --print-architecture) \
      signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
      https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
    RUN apt-get update && apt-get install -y docker-ce-cli
    USER jenkins
    RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
    
    ---------------------------------------------------------------------------------
    
    docker build -t myjenkins-blueocean:2.387.3-1 .
    
    docker run \
      --name jenkins-blueocean \
      --restart=on-failure \
      --detach \
      --network jenkins \
      --env DOCKER_HOST=tcp://docker:2376 \
      --env DOCKER_CERT_PATH=/certs/client \
      --env DOCKER_TLS_VERIFY=1 \
      --publish 8080:8080 \
      --publish 50000:50000 \
      --volume jenkins-data:/var/jenkins_home \
      --volume jenkins-docker-certs:/certs/client:ro \
      myjenkins-blueocean:2.387.3-1
    
    docker exec -it contianerId bash
    
    # Jenkins Container 에서 도커 테스트
    docker run hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    719385e32844: Pull complete 
    Digest: sha256:fc6cf906cbfa013e80938cdf0bb199fbdbb86d6e3e013783e5a766f50f5dbce0
    Status: Downloaded newer image for hello-world:latest
    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    ```
    
- AWS EC2 보안그룹 인바운드 규칙 추가
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b0113a1b-23f0-4fa5-8069-4e7b811c3481/Untitled.png)
    
- Jenkins 컨테이너 접속 확인
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5f9f41ca-53c5-4e17-9b8c-b0fb52d18094/Untitled.png)
    
- Jenkins 환경 설정
    - 빌드에 필요한 플러그인 및 패키지 설치
    - git, ant, docker 확인
    
    ```bash
    apt install ant
    ant -v
    Apache Ant(TM) version 1.10.9 compiled on December 25 1969
    ```
    

---

### Argo CD컨테이너 실행 및 Item 설정

- argocd namespace 생성
    
    ```bash
    kubectl create namespace argocd
    ```
    
- Argo CD 컨테이너 실행
    
    ```bash
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    
    kubectl get svc,pod -n argocd
    NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    service/argocd-applicationset-controller          ClusterIP   10.108.203.119   <none>        7000/TCP,8080/TCP            94s
    service/argocd-dex-server                         ClusterIP   10.100.91.72     <none>        5556/TCP,5557/TCP,5558/TCP   94s
    service/argocd-metrics                            ClusterIP   10.103.98.125    <none>        8082/TCP                     94s
    service/argocd-notifications-controller-metrics   ClusterIP   10.107.124.8     <none>        9001/TCP                     93s
    service/argocd-redis                              ClusterIP   10.104.177.50    <none>        6379/TCP                     93s
    service/argocd-repo-server                        ClusterIP   10.100.10.177    <none>        8081/TCP,8084/TCP            93s
    service/argocd-server                             ClusterIP   10.101.95.43     <none>        80/TCP,443/TCP               93s
    service/argocd-server-metrics                     ClusterIP   10.99.18.8       <none>        8083/TCP                     93s
    
    NAME                                                    READY   STATUS    RESTARTS   AGE
    pod/argocd-application-controller-0                     1/1     Running   0          93s
    pod/argocd-applicationset-controller-6bf657d56d-694zg   1/1     Running   0          93s
    pod/argocd-dex-server-5778f5b7b4-tbjqb                  1/1     Running   0          93s
    pod/argocd-notifications-controller-5d49c977f5-jpj92    1/1     Running   0          93s
    pod/argocd-redis-77bf5b886-cv8wm                        1/1     Running   0          93s
    pod/argocd-repo-server-7bd9b7df75-pbksk                 1/1     Running   0          93s
    pod/argocd-server-f84cf8b46-j2fqb                       1/1     Running   0          93s
    ```
    
- 서비스 NodePort 로 변경
    
    ```bash
    # 외부에서 접속할 수 있도록 NodePort 타입으로 변경
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
    kubectl get svc argocd-server -n argocd -w
    NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    argocd-server   ClusterIP   10.101.95.43   <none>        80/TCP,443/TCP   2m32s
    argocd-server   NodePort    10.101.95.43   <none>        80:32383/TCP,443:30827/TCP   5m16s
    ```
    
- AWS EC2 보안그룹 인바운드 규칙 추가
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/95e815fc-9b21-463a-90a2-51fc8a968266/Untitled.png)
    
- Argo CD 접속 테스트
    
    ```bash
    # default 계정 admin
    # 패스워드 명령어
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4fb3c02e-c71e-4ee8-93fb-2319f6ba5ed1/Untitled.png)
    
- Argo CD Item 추가
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/799ea007-723b-45ce-a5fa-62bb35353a8e/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d7c05388-9ddb-451b-a1b4-61d63c891f6c/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2f22ca32-3ae6-40d5-9f29-4840ce1ccc5a/Untitled.png)
    

---

### gitea 컨테이너 실행 및 repo 설정

- gitea  컨테이너 실행
    
    ```bash
    sudo docker run -d --name=gitea -p 3000:3000 -v /var/lib/gitea:/data gitea/gitea:latest
    
    sudo docker ps
    CONTAINER ID   IMAGE                COMMAND                   CREATED          STATUS         PORTS                                                                                      NAMES
    105deae4e7a8   gitea/gitea:latest   "/usr/bin/entrypoint…"   10 seconds ago   Up 9 seconds   22/tcp, 0.0.0.0:3000->3000/tcp, :::3000->3000/tcp                                          gitea
    53f1701b2312   jenkins/jenkins      "/usr/bin/tini -- /u…"   2 hours ago      Up 2 hours     0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:50000->50000/tcp, :::50000->50000/tcp   jenkins-server
    ```
    
- AWS EC2 보안그룹 인바운드 규칙 추가
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e62ed914-a570-4e5a-9eeb-be0ab10390c7/Untitled.png)
    
- gitea 접속 테스트
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6fb6ff0c-f097-4c3c-a2e1-c65b266c571b/Untitled.png)
    

---

### gitea repo 생성

- 초기 설정
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/89071834-061e-43cd-9454-018aa99b8390/Untitled.png)
    
- tomcat-project repo 생성
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/86be14be-3186-4e03-b657-241b238a958b/Untitled.png)
    
    ```bash
    mkdir tomcat-project
    touch README.md
    git init
    git checkout -b main
    git add README.md
    git config --global user.email "won5854@naver.com"
    git config --global user.name "hyewon"
    git commit -m "first commit"
    git remote add origin http://13.124.203.239:3000/montoring/tomcat-project.git
    git push -u origin main
    ```
    
    ```bash
    scp -i hw_jenkins.pem apache-tomcat-8.5.88-src.tar.gz centos@13.124.203.239:/home/centos/tomcat-project
    ```
    
    ```bash
    ls
    README.md  apache-tomcat-8.5.88-src.tar.gz
    
    gunzip apache-tomcat-8.5.88-src.tar.gz 
    ls
    README.md  apache-tomcat-8.5.88-src.tar
    
    tar xf apache-tomcat-8.5.88-src.tar 
    ls
    README.md  apache-tomcat-8.5.88-src  apache-tomcat-8.5.88-src.tar
    
    mv apache-tomcat-8.5.88-src/* .
    ls
    BUILDING.txt     README.md                     build.properties.default  res
    CONTRIBUTING.md  RELEASE-NOTES                 build.properties.release  test
    KEYS             RUNNING.txt                   build.xml                 webapps
    LICENSE          apache-tomcat-8.5.88-src      conf
    MERGE.txt        apache-tomcat-8.5.88-src.tar  java
    NOTICE           bin                           modules
    
    rm -rf apache-tomcat-8.5.88-src
    rm -rf apache-tomcat-8.5.88-src.tar 
    ls
    BUILDING.txt     MERGE.txt      RUNNING.txt               build.xml  res
    CONTRIBUTING.md  NOTICE         bin                       conf       test
    KEYS             README.md      build.properties.default  java       webapps
    LICENSE          RELEASE-NOTES  build.properties.release  modules
    
    git add --all
    git commit -m "tomcat unzip commit"
    ```
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5f165e14-d9c5-4589-a038-dce4a0a521f2/Untitled.png)
    
- argocd-yaml repo 생성
    - deployment.yaml, service-nodeport.yaml 파일 생성
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e516310d-cc99-4551-95c1-39159049113f/Untitled.png)
    
    ```bash
    sudo yum install git
    
    touch README.md
    git init
    git checkout -b main
    git add README.md
    git config --global user.email "won5854@naver.com"
    git config --global user.name "hyewon"
    git commit -m "first commit"
    git remote add origin http://13.124.203.239:3000/montoring/argocd-yaml.git
    git push -u origin main
    ```
    
    ```bash
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: tomcat-deployment
      labels:
        argocd: deploy
    spec:
      replicas: 2
      selector:
        matchLabels:
          argocd: deploy
      template:
        metadata:
          labels:
            argocd: deploy
        spec:
          containers:
          - name: tomcat
            image: hyewone/mentoring_project:latest
            ports:
            - containerPort: 80
    
    ---
    
    apiVersion: v1
    kind: Service
    metadata:
      name: service-nodeport
    spec:
      type: NodePort
      selector:
        argocd: deploy
      ports:
        - port: 80
          targetPort: 8080
          nodePort: 31111
    ```
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e639c4f7-d766-447b-9845-c801515a3245/Untitled.png)
    
- 노드 식별 라벨 생성
    
    ```bash
    k label node master argoCD=deploy
    
    k get node -L argoCD
    NAME     STATUS   ROLES           AGE     VERSION   ARGOCD
    master   Ready    control-plane   8m42s   v1.27.1   deploy
    ```
    

---

### 도커 레포지토리 생성 및 Dockerfile 작성

- 도커 레포지토리 생성
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4badfe99-b777-4798-a656-81b5dd6b2c9d/Untitled.png)
    
- Dockerfile 작성
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/24b0b52c-dfb5-4906-a374-3fd67d1a55b0/Untitled.png)
    

---

### Jenkins Pipeline 작성

```groovy
pipeline {
    agent any
    
    environment {
        GITEA_DOMAIN = "13.124.203.239:3000/montoring/"
        APP_REPO = "tomcat-project.git"
        YAML_REPO = "argocd-yaml.git"
        DOCKER_IMAGE = "hyewone/mentoring_project"
        DOCKER_IMAGE_TAG = "v${env.BUILD_ID}"
    }
    
    stages {
       
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                dir('app-repo') {
                    git branch: 'main', url: 'http://' + GITEA_DOMAIN + APP_REPO 
               
                    sh 'ant clean'
                    sh 'ant package'
                    sh 'ant deploy'
                }
            }
        }
        
        stage('Upload to Docker Registry') {
            steps {
                // 도커 이미지 빌드 및 업로드
                dir('app-repo') {
                    
                    withCredentials([usernamePassword(credentialsId: 'docker_login_info', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} ."
                        sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }
        
        stage('Update ArgoCD YAML') {
            steps {
                // ArgoCD YAML 파일 체크아웃
                dir('yarml-repo') {
                    
                    git branch: 'main', url: 'http://' + GITEA_DOMAIN + YAML_REPO
                                
                    script {
                        // YAML 파일에서 이미지 버전 변경
                        sh 'sed -i "s|' + DOCKER_IMAGE +':.*|' + DOCKER_IMAGE + ':' + DOCKER_IMAGE_TAG +'|" deployment.yaml'
                    }
                   
                    // 변경된 YAML 파일 커밋 및 푸시
                    withCredentials([usernamePassword(credentialsId: 'git_email', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'git config user.email ${USERNAME}'
                    }
                    withCredentials([usernamePassword(credentialsId: 'git_name', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'git config user.name ${USERNAME}'
                    }
                    sh 'git add deployment.yaml'
                    sh 'git commit -m "Update image version"'
                    
                    withCredentials([usernamePassword(credentialsId: 'gitea_token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'git push http://${USERNAME}:${PASSWORD}@${GITEA_DOMAIN}${YAML_REPO}'
                    }
                }
            }
        }
    }

    post {
        failure {
            slackSend channel: '#k8s-cicd',
            color: 'danger',
            message: "${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} Build failed!",
            tokenCredentialId: 'slack_webhook'
        }

        success {
            slackSend channel: '#k8s-cicd',
            color: 'good',
            message: "${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} Build successful!",
            tokenCredentialId: 'slack_webhook'
        }
    }
}
```

- agent any : build 하는 agent
- environment :
- stage('Build') : ant build 단계
- stage('Upload to Docker Registry') : 도커 이미지 생성 및 도커 이미지 업로드 단계
- stage('Update ArgoCD YAML') : 도커 레포지토리와 이미지 버전 비교하여, yaml 파일의 버전을 최신 버전으로 update commit

---

### Jenkins Pipeline 테스트

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/840cfdf4-75cf-47ab-a7d9-9c57e3dc3100/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eb17b8d1-75ab-456b-af0d-6a9ca2ecac34/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d64ffaa8-bf23-48cf-82f9-c65b18ce9a18/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b15c034-ec8b-4e57-8149-0a4b6ba0d0f2/Untitled.png)

```bash
k get pod,svc -n argocd-deploy 
NAME                                     READY   STATUS    RESTARTS   AGE
pod/tomcat-deployment-8679c89c8f-2nkcp   1/1     Running   0          16m
pod/tomcat-deployment-8679c89c8f-55vf7   1/1     Running   0          16m

NAME                       TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/service-nodeport   NodePort   10.98.43.62   <none>        80:31111/TCP   19h
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f9024e7a-2b51-4e2e-a87f-996d44c00b9f/Untitled.png)

---

### Webhook 설정

- Jenkins Generic Webhook Trigger Plugin 설치
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29365104-6854-46f9-a6f0-b07cf436aa58/Untitled.png)
    
- Jenkins API Token 발급
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/910a8fac-883c-472f-8a90-9515d2ad07cf/Untitled.png)
    
- gitea Webhook  설정
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8b078d7b-823b-403a-9dd4-93e8d218d2ca/Untitled.png)
    
- Jenkins 프로젝트 Generic Webhook Trigger 설정
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dc865380-5d6e-4706-8644-7d4fbef71b56/Untitled.png)
    
- Webhook 전달 시험
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/757d5151-3f2e-4018-aa21-2b47f185274c/Untitled.png)
    
- build
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c697b8f3-d942-467b-a380-6b5180d95f4c/Untitled.png)
    

- 
    
    ```bash
    
    # Jenkins Build 성공 후 Jenkins Server에 실제로 빌드된 파일 확인
    
    ls -al /var/jenkins_home/workspace/tomcat-project
    total 348
    drwxr-xr-x. 11 root root   4096 May 11 13:06 .
    drwxr-xr-x.  4 root root     54 May 11 12:57 ..
    drwxr-xr-x.  8 root root    162 May 11 13:06 .git
    -rw-r--r--.  1 root root  20233 May 11 12:57 BUILDING.txt
    -rw-r--r--.  1 root root   6210 May 11 12:57 CONTRIBUTING.md
    -rw-r--r--.  1 root root  44901 May 11 12:57 KEYS
    -rw-r--r--.  1 root root  57011 May 11 12:57 LICENSE
    -rw-r--r--.  1 root root   3225 May 11 12:57 MERGE.txt
    -rw-r--r--.  1 root root   1726 May 11 12:57 NOTICE
    -rw-r--r--.  1 root root   3398 May 11 12:57 README.md
    -rw-r--r--.  1 root root   7231 May 11 12:57 RELEASE-NOTES
    -rw-r--r--.  1 root root  16745 May 11 12:57 RUNNING.txt
    drwxr-xr-x.  2 root root   4096 May 11 12:57 bin
    -rw-r--r--.  1 root root  16159 May 11 12:57 build.properties.default
    -rw-r--r--.  1 root root   2209 May 11 12:57 build.properties.release
    -rw-r--r--.  1 root root 149429 May 11 12:57 build.xml
    drwxr-xr-x.  2 root root    238 May 11 12:57 conf
    drwxr-xr-x.  4 root root     30 May 11 12:57 java
    drwxr-xr-x.  3 root root     23 May 11 12:57 modules
    drwxr-xr-x.  7 root root     80 May 11 13:07 output
    drwxr-xr-x. 11 root root    198 May 11 12:57 res
    drwxr-xr-x. 24 root root   4096 May 11 12:57 test
    drwxr-xr-x.  7 root root     81 May 11 12:57 webapps
    
    # 도커파일 테스트를 위해 먼저 jenkins server에서 cli로 테스트
    mkdir temp/
    cd temp/
    git clone http://13.124.203.239:3000/montoring/tomcat-project.git
    Cloning into 'tomcat-project'...
    remote: Enumerating objects: 4493, done.
    remote: Counting objects: 100% (4493/4493), done.
    remote: Compressing objects: 100% (2568/2568), done.
    remote: Total 4493 (delta 1802), reused 4487 (delta 1802), pack-reused 0
    Receiving objects: 100% (4493/4493), 12.98 MiB | 16.27 MiB/s, done.
    Resolving deltas: 100% (1802/1802), done.
    
    cd tomcat-project/
    ls
    BUILDING.txt	 KEYS	  MERGE.txt  README.md	    RUNNING.txt  build.properties.default  build.xml  java     res   webapps
    CONTRIBUTING.md  LICENSE  NOTICE     RELEASE-NOTES  bin		 build.properties.release  conf       modules  test
    vim Dockerfile
    
    # 베이스 이미지 선택
    FROM openjdk:11
    
    # 작업 디렉토리 생성
    WORKDIR /usr/local/tomcat/
    
    # JAR 파일을 웹 애플리케이션 디렉토리에 복사
    COPY output/build/ /usr/local/tomcat/
    
    CMD ["/usr/local/tomcat/bin/catalina.sh", "run"]
    
    docker build -t hyewone/mentoring_project:latest .
    docker run -p 3333:8080 --name dockerfile-test hyewone/mentoring_project
    
    ```
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9b8669ff-7cd8-4d79-ac7e-3ac98eb60395/Untitled.png)
    
    ```bash
    docker ps
    CONTAINER ID   IMAGE                       COMMAND                  CREATED         STATUS         PORTS                    NAMES
    1bcd48d6f4d5   hyewone/mentoring_project   "/usr/local/tomcat/b…"   4 minutes ago   Up 4 minutes   0.0.0.0:3333->8080/tcp   dockerfile-test
    ```
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8a5789ba-9852-415b-8073-0796538569c7/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9c68264a-d722-4f4d-a216-b50e6af819a9/Untitled.png)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49b02727-9dfd-4694-8783-59b1d88ce6c9/Untitled.png)
    
    ```python
    apt install docker.io
    service docker start
    Starting Docker: docker.
    ```
    
    ```bash
    service docker status
    Docker is not running ... failed!
    
    tail -f /var/log/docker.log 
    time="2023-05-11T13:30:04.827772754Z" level=warning msg="Unable to setup quota: operation not permitted\n"
    time="2023-05-11T13:30:04.831967072Z" level=info msg="Loading containers: start."
    time="2023-05-11T13:30:04.832173984Z" level=warning msg="Running modprobe bridge br_netfilter failed with message: , error: exec: \"modprobe\": executable file not found in $PATH"
    time="2023-05-11T13:30:04.834124924Z" level=warning msg="Running iptables --wait -t nat -L -n failed with message: `# Warning: iptables-legacy tables present, use iptables-legacy to see them\niptables v1.8.7 (nf_tables): Could not fetch rule set generation id: Permission denied (you must be root)`, error: exit status 4"
    time="2023-05-11T13:30:04.858705051Z" level=info msg="stopping event stream following graceful shutdown" error="<nil>" module=libcontainerd namespace=moby
    time="2023-05-11T13:30:04.859047893Z" level=info msg="stopping healthcheck following graceful shutdown" module=libcontainerd
    time="2023-05-11T13:30:04.859053231Z" level=info msg="stopping event stream following graceful shutdown" error="context canceled" module=libcontainerd namespace=plugins.moby
    failed to start daemon: Error initializing network controller: error obtaining controller instance: failed to create NAT chain DOCKER: iptables failed: iptables -t nat -N DOCKER: iptables v1.8.7 (nf_tables): Could not fetch rule set generation id: Permission denied (you must be root)
    
    # root 사용자임에도..
    iptables -L
    # Warning: iptables-legacy tables present, use iptables-legacy to see them
    iptables v1.8.7 (nf_tables): Could not fetch rule set generation id: Permission denied (you must be root)
    ```
    
    ---
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7e78558f-35c4-449b-a0a8-885922c6f42f/Untitled.png)
    
    ```bash
    sudo docker exec -it 554cc006c498 bash
    apt update
    apt install maven
    
    mvn -v
    Apache Maven 3.6.3
    Maven home: /usr/share/maven
    Java version: 11.0.19, vendor: Eclipse Adoptium, runtime: /opt/java/openjdk
    Default locale: en, platform encoding: UTF-8
    OS name: "linux", version: "3.10.0-1160.76.1.el7.x86_64", arch: "amd64", family: "unix"
    ```
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7933d9b8-23a2-421f-a074-38e4e5949b24/Untitled.png)
    
    ```bash
    kubectl get po -A
      764  k get node
      765  k get sc -A
      766  ls
      767  cd tomcat-project/
      768  ls
      769  exit
      770  k get po -A
      771  exit
      772  docker ps
      773  sudosu
      774  sudo su
      775  rancher
      776  k get po -A
      777  k get node
      778  history | grep taint
      779  k top po -A
      780  sudo su
      781  clear
      782  sudo docker ps
      783  k get pod -A
    ```
