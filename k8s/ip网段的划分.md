
--service-cluster-ip-range  
- 集群中 Service 对象的 CIDR 范围(默认IP段 10.43.0.0/16  )
- kube-apiserver and kube-controller-manager

--cluster-cidr
- 集群中 Pods 的 CIDR 范围

这两个IP段没有任何的关联,不需要谁包含谁的情况.只要保证两个ip段的地址不会冲突就可以

建议:
- 为了避免冲突,建议使用两个不同的网段,比如server的ip段使用172.16.0.0/16,pod的子网使用10.10.0.0/16
