# homelab-k8s

自宅 k8s 環境(Proxmox 上の kubeadm クラスター)で使っているマニフェストと手順の覚書。
自分の環境に合わせた内容なので、そのままでは動かない。

Ansible のほうは [homelab-ansible](https://github.com/sugerara/homelab-ansible) に置いている。

## 構成

```
.
├── README.md
├── .gitignore
└── storage/
    └── storageclass-nfs-csi.yaml
```

## storage

NFS サーバー(acorn)を、csi-driver-nfs で動的プロビジョニングに使っている。

### csi-driver-nfs のインストール

```
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm repo update
helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs \
  --namespace kube-system \
  --version 4.13.4
```

### StorageClass の適用

```
kubectl apply -f storage/storageclass-nfs-csi.yaml
```

`nfs-csi` をデフォルトの StorageClass にしている。
最初から入っていた `local-path` は、デフォルトから外した。

```
kubectl patch storageclass local-path \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

### メモ

- NFS のバージョンは 4.2(`mountOptions` の `nfsvers=4.2`)。
- reclaimPolicy は Delete。PVC を消すと、NFS 上のディレクトリも消える。
- 差分の確認は `kubectl diff -f storage/`。

## 注意

- kubeconfig、鍵、Secret は置かない(`.gitignore` で除外している)。
