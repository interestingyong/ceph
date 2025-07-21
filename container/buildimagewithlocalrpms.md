 mkdir -p /etc/systemd/system/docker.service.d/
 vim /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.1.202:8889"
Environment="HTTPS_PROXY=http://192.168.1.202:8889"

tail -n 3 Dockerfile.build
    SCCACHE_URL="https://github.com/mozilla/sccache/releases/download/v${SCCACHE_VERSION}/sccache-v${SCCACHE_VERSION}-$(uname -m)-unknown-linux-musl.tar.gz"; \
    export https_proxy=192.168.1.202:8889; \
    curl -L $SCCACHE_URL | tar --no-anchored --strip-components=1 -C /usr/local/bin/ -xzf - sccache

systemctl daemon-reload
systemctl restart docker
 ./src/script/build-with-container.py --image-repo=quay.io/ceph-ci/ceph --tag=main --image-sources=build
rm -rf build;
 ./src/script/build-with-container.py   -d centos9 -e build 在容器中编译
 docker build --pull -t ceph-build:main.centos9 --label=io.ceph.build-with-container.src=sha256:899343471d106472633c204cc68097ea4b161ef1d45f216049ffc1f490690ee9 --build-arg=CEPH_BASE_BRANCH=main --build-arg=DISTRO=quay.io/centos/centos:stream9 -f Dockerfile.build .

 ps -ef|grep -i docker
root       99065       1  1 16:31 ?        00:00:27 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root      105658  105633  0 16:48 pts/0    00:00:00 docker build --pull -t ceph-build:main.centos9 --label=io.ceph.build-with-container.src=sha256:899343471d106472633c204cc68097ea4b161ef1d45f216049ffc1f490690ee9 --build-arg=CEPH_BASE_BRANCH=main --build-arg=DISTRO=quay.io/centos/centos:stream9 -f Dockerfile.build .
root      105695  105658  0 16:48 pts/0    00:00:09 /usr/libexec/docker/cli-plugins/docker-buildx buildx build --pull -t ceph-build:main.centos9 --label=io.ceph.build-with-container.src=sha256:899343471d106472633c204cc68097ea4b161ef1d45f216049ffc1f490690ee9 --build-arg=CEPH_BASE_BRANCH=main --build-arg=DISTRO=quay.io/centos/centos:stream9 -f Dockerfile.build .
root      105756   99065  0 16:48 ?        00:00:00 runc --log /var/lib/docker/buildkit/executor/runc-log.json --log-format json run --bundle /var/lib/docker/buildkit/executor/fvpd6kig777b9swqd5jtg6x6c --keep fvpd6kig777b9swqd5jtg6x6c
root      107801   38405  0 17:15 pts/1    00:00:00 grep --color=auto -i docker

rook目录文件被谁清理了
[图片]
cd ceph/src/pybind/mgr/rook/rook-client-python
git reset --hard
/ceph/src/breakpad/src/client/linux/crash_generation/crash_generation_client.cc:43:10: fatal error: third_party/lss/linux_syscall_support.h: No such file or directory
这个目录也是需要git reset

打包镜像
前面只是编译，默认container/dockerfile是下载服务端的rpm包来实现build。并没有考虑本地镜像的情况。
所以如果你能手动build，push过去是可以的。如果不行的话。可以考虑自己建一个rpm站。
或者还有其他方式：
- 生成rpm包，并在dockerfile里面进行手动安装。
- ./src/script/build-with-container.py -d centos9 -e rpm
- 实现dockerfile， 手动构建ceph目录。把文件从build目录拷贝过去。
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-base-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/librgw2-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/noarch/ceph-mgr-dashboard-20.3.0-1599.g614e83cfecc.el9.noarch.rpm
Wrote: /ceph/rpmbuild/RPMS/noarch/ceph-mgr-diskprediction-local-20.3.0-1599.g614e83cfecc.el9.noarch.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-mgr-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-base-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-radosgw-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/rbd-mirror-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/librados2-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-mds-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-mon-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/librbd1-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-debugsource-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/librgw2-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-common-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-test-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-osd-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-radosgw-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-osd-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-common-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
Wrote: /ceph/rpmbuild/RPMS/x86_64/ceph-test-debuginfo-20.3.0-1599.g614e83cfecc.el9.x86_64.rpm
创建一个local repo，从本地安装ceph rpm文件。其他保留。
apt install podman -y
cd ceph/Container
mkdir localrpms
cp ../rpmbuild/RPMS/x86_64/* ./localrpms/
cp ../rpmbuild/RPMS/noarch/* ./localrpms/
apt install createrepo -y
cd ./localrpms ;  createrepo-c .
cd ../
 ./build.sh
 生成的镜像是再podman里面隔离的。 需要导入到docker里面测试。
 podman images
REPOSITORY                          TAG                IMAGE ID      CREATED         SIZE
localhost/build.sh.output           latest             91e8abd4cdce  25 minutes ago  5.08 GB
quay.ceph.io/ceph/prerelease-amd64  vv20.3.0-20250721  91e8abd4cdce  25 minutes ago  5.08 GB
quay.io/centos/centos               stream9            ab9123964b93  4 weeks ago     171 MB

podman save -o ceph-image.tar quay.ceph.io/ceph/prerelease-amd64:vv20.3.0-20250721
docker load -i ceph-image.tar  
docker tag quay.ceph.io/ceph/prerelease-amd64:vv20.3.0-20250721 quay.ceph.io/ceph-ci/ceph:main
podman tag quay.ceph.io/ceph/prerelease-amd64:vv20.3.0-20250721 quay.ceph.io/ceph-ci/ceph:main  （cephadm的镜像来自podman？ 需要确认下. 其实可能只有这步需要，如果其实用的是podman部署）

docker images
REPOSITORY                           TAG                 IMAGE ID       CREATED          SIZE
quay.ceph.io/ceph-ci/ceph            main                e7695ab6e857   18 minutes ago   5.05GB
quay.ceph.io/ceph/prerelease-amd64   vv20.3.0-20250721   e7695ab6e857   18 minutes ago   5.05GB

