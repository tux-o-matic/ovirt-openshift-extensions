# Copyright 2018 oVirt-maintainers
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM centos:7.6.1810

ARG VERSION
ARG RELEASE

LABEL   com.redhat.component="ovirt-openshift-extensions" \
        name="ovirt-flexvolume-driver" \
        version="$VERSION" \
        release="$RELEASE" \
        architecture="x86_64" \
        summary="ovirt-flexvolume-driver for openshift, for dynamic volume provisioning" \
        maintainer="Roy Golan <rgolan@redhat.com>"

RUN mkdir -p /root/go/src/github.com/ovirt/ovirt-openshift-extensions
WORKDIR /root/go/src/github.com/ovirt/ovirt-openshift-extensions
ADD ovirt-openshift-extensions-$VERSION-$RELEASE.tar.gz .

RUN yum install -y epel-release --enablerepo=extras && \
    yum install -y --enablerepo=extras --disablerepo=updates golang make &&  \
    make ovirt-flexvolume-driver && \
    mv ovirt-flexvolume-driver /usr/bin/ && \
    yum history -y rollback 2 && \ 
    yum clean all && rm -rf /var/cache/yum


#TODO use multi-stage when buildah runs in mock or docker 1.17 is available
RUN cp -v deployment/ovirt-flexvolume-driver/entrypoint.sh /

ENTRYPOINT ["/entrypoint.sh"]
