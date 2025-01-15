# ☸️ Ansible Kubernetes Playbook

이 프로젝트는 Ansible을 사용해 **Kubernetes 클러스터**(마스터/워커 노드) 및 **NFS 환경**을 자동으로 구성하는 플레이북입니다.
주요 기능은 다음과 같습니다.

- 🚀 **Kubernetes 기본 환경 설치** (컨테이너 런타임, kubeadm, kubelet, kubectl 등)
- ⚙️ **클러스터 초기화** 및 Calico 네트워크 플러그인 배포
- 📁 **NFS 서버/클라이언트** 설치 및 공유 디렉터리 설정

## 📂 프로젝트 구조

.
├── LICENSE
├── README.md            # (바로 이 파일)
├── ansible.cfg
├── inventory
│   ├── host_vars
│   │   ├── master.yml
│   │   ├── worker1.yml
│   │   ├── worker2.yml
│   │   └── worker3.yml
│   └── inventory.ini
├── k8s_playbook.yml     # 플레이북 (마스터/워커 각각 따로 구성)
└── roles
├── init_k8s         # 클러스터 초기화(마스터 역할)
├── install_k8s      # 쿠버네티스 설치 및 기본 설정(마스터/워커 공통)
└── nfs              # NFS 서버 설치/공유

## 📝 사용 방법

1. **사전 준비**
   - Ansible 설치가 완료된 제어 노드(Control Node)가 있어야 합니다.
   - `inventory/inventory.ini` 파일에서 마스터, 워커 노드를 정의합니다.

     ```ini
     [masters]
     master ansible_host=xxx.xxx.xxx.xxx

     [workers]
     worker1 ansible_host=yyy.yyy.yyy.yyy
     worker2 ansible_host=zzz.zzz.zzz.zzz
     ...
     ```

   - 필요하다면 `inventory/host_vars`에 호스트별 변수를 세부적으로 정의합니다.

2. **Ansible 플레이북 실행**

   ```bash
   ansible-playbook -i inventory/inventory.ini k8s_playbook.yml

- 플레이북은 먼저 워커 노드 쪽에 기본 설치(`install_k8s`)를 수행하고, 이후 마스터 노드(또는 `masters` 그룹)에서 초기화(`init_k8s`)와 클러스터 검증을 진행합니다.

**Ansible 조건**

- ansible core.ver : 2.16.12
- python version : 3.12

1. **플레이북 완료 후**
    - 마스터 노드에서 `kubectl get nodes` 명령으로 클러스터 노드가 `Ready` 상태인지 확인 가능합니다.
    - NFS 서버가 마스터 노드(또는 특정 노드)에서 동작하며, 워커 노드에는 NFS 클라이언트가 설치되어 공유 디렉터리를 마운트할 수 있습니다.

## 🔑 주요 변수

- **Kubernetes 관련**:
  - `k8s_version`: 쿠버네티스 버전 태그 (예: `v1.32`)
  - `Pod_CIDR`: 파드 네트워크 CIDR (예: `172.16.0.0/24`)
  - `Calico_version`: Calico 설치 버전 (예: `v3.28.1`)
- **NFS 관련**:
  - `nfs_shared_dir_path`: NFS 공유 디렉터리 경로 (예: `/srv/nfsv4/shared`)
  - `cluster_node_CIDR`: 접근할 클라이언트 CIDR (예: `192.168.0.0/16`)
  - `subDirPath`: 공유 폴더 하위 디렉토리 목록 (예: `['jenkins_home', 'vault', ...]`)

이 외에도 롤별 `vars`나 `defaults` 디렉토리를 참고해보시면 더 다양한 파라미터를 확인하실 수 있습니다.

## 🤝 기여 방법

1. 이 저장소를 포크(Fork) 후 로컬로 클론(clone)합니다.
2. 새로운 브랜치를 생성하고(`git checkout -b feature/my-improvement`) 수정 사항을 작업합니다.
3. 변경 사항을 커밋하고(`git commit -m "Add some improvements"`) 푸시합니다.
4. Pull Request를 생성하여 변경 사항을 공유합니다.

## 🎉 마치며

이 플레이북을 통해 **간단하고 효율적으로** Kubernetes 클러스터를 구축할 수 있습니다.

버그나 개선점은 이슈(issues)나 PR을 통해 언제든 제보 부탁드립니다. 감사합니다. 🙏
