

# 1. Скачай утилиту etcdctl, если у тебя ее еще нет.

Reference: https://github.com/etcd-io/etcd/releases

```
ETCD_VER=v3.4.9

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
ETCDCTL_API=3 /tmp/etcd-download-test/etcdctl version

mv /tmp/etcd-download-test/etcdctl /usr/bin
```

# 2. Сними копию

```
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
     snapshot save /opt/snapshot-pre-boot.db
```

Внимание: В этом случае, **ETCD** запущена на том же сервере, где мы запускаем команды (т.е. у нас это узел *controlplane*). В этом случае нам не нужна опция **--endpoint**, поэтому ее можно игнорировать. 

Опции **--cert, --cacert and --key** обязательны для аутентификации на сервере ETCD, чтобы сделать резервную копию.

Если вы хотите сделать резервную копию службы ETCD, работающей на другом компьютере, то нужно будет предоставить правильную конечную точку для этого сервера (это IP-адрес и порт сервера ETCD с аргументом **--endpoint**)

# -----------------------------
# После аварии
# -----------------------------

# 3. Восстанови ETCD snapshot в новую папку

```
ETCDCTL_API=3 etcdctl  --data-dir /var/lib/etcd-from-backup \
     snapshot restore /opt/snapshot-pre-boot.db
```

Внимание: В этом случае мы восстанавливаем snapshot в другой каталог, но на том же сервере, где мы делали резервную копию (**controlplane**)
В результате единственной обязательной опцией для команды восстановления является **--data-dir**.

# 4. Измени /etc/kubernetes/manifests/etcd.yaml

Мы восстановили ETCD snapshot по новому пути на узле controlplane - **/var/lib/etcd-from-backup**, поэтому единственное изменение, которое нужно сделать в файле YAML, - это изменить hostPath для тома с именем **etcd-data**. Т.е. меняем старый каталог **/var/lib/etcd** на новый  **/var/lib/etcd-from-backup**.

```
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
```
После этого изменения, /var/lib/etcd в **контейнере** с ETCD будет указывать в /var/lib/etcd-from-backup на узле **controlplane**.

После обновления файла-манифеста, POD, содержаший ETCD, автоматически пересоздастся, т.к. это **static POD**, лежащий в размещении `/etc/kubernetes/manifests`.


> Примечание: поскольку POD ETCD был изменен, он автоматически перезапустится, а также kube-controller-manager и kube-scheduler. Подожди 1-2 минуты, чтобы эти PODs перезапустились. Можно сделать `watch docker ps | grep etcd `, чтобы увидеть, когда POD с ETCD перезапустится.

> Примечание 2: если POD ETCD не достигает `Ready 1/1`, перезапусти его с помощью `kubectl delete pod -n kube-system etcd-controlplane` и подожди минуту.

> Примечание 3: есть простой способ убедиться, что ETCD использует восстановленные данные после воссоздания контейнера ETCD. Тебе **не** нужно ничего менять.
  
   **Если** ты изменишь **--data-dir** на **/var/lib/etcd-from-backup** в файле YAML, убедись, что **volumeMounts** для **etcd-data** также обновился и **mountPath** указывает на `/var/lib/etcd-from-backup` (**ЭТОТ ШАГ ЯВЛЯЕТСЯ ДОПОЛНИТЕЛЬНЫМ И НЕ ТРЕБУЕТСЯ ДЛЯ ЗАВЕРШЕНИЯ ВОССТАНОВЛЕНИЯ**)
