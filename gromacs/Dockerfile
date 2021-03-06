# Build stage with Spack pre-installed and ready to be used
FROM spack/ubuntu-bionic:latest as builder

# Install OS packages needed to build the software
RUN apt-get -yqq update && apt-get -yqq upgrade \
 && apt-get -yqq install cmake perl \
 && rm -rf /var/lib/apt/lists/*

# What we want to install and how we want to install it
# is specified in a manifest file (spack.yaml)
RUN mkdir /opt/spack-environment \
&&  (echo "spack:" \
&&   echo "  specs:" \
&&   echo "  - gromacs+mpi target=broadwell" \
&&   echo "  - openmpi" \
&&   echo "  packages:" \
&&   echo "    perl:" \
&&   echo "      externals:" \
&&   echo "      - spec: perl@5.26.1" \
&&   echo "        prefix: /usr" \
&&   echo "      buildable: false" \
&&   echo "    cmake:" \
&&   echo "      externals:" \
&&   echo "      - spec: cmake@3.10.2" \
&&   echo "        prefix: /usr" \
&&   echo "      buildable: false" \
&&   echo "  concretization: together" \
&&   echo "  config:" \
&&   echo "    install_tree: /opt/software" \
&&   echo "  view: /opt/view") > /opt/spack-environment/spack.yaml

# Install the software, remove unnecessary deps
RUN cd /opt/spack-environment && spack env activate . && spack install --fail-fast && spack gc -y

# Strip all the binaries
RUN find -L /opt/view/* -type f -exec readlink -f '{}' \; | \
    xargs file -i | \
    grep 'charset=binary' | \
    grep 'x-executable\|x-archive\|x-sharedlib' | \
    awk -F: '{print $1}' | xargs strip -s

# Modifications to the environment that are necessary to run
RUN cd /opt/spack-environment && \
    spack env activate --sh -d . >> /etc/profile.d/z10_spack_environment.sh


# Bare OS image to run the installed executables
FROM ubuntu:18.04

COPY --from=builder /opt/spack-environment /opt/spack-environment
COPY --from=builder /opt/software /opt/software
COPY --from=builder /opt/view /opt/view
COPY --from=builder /etc/profile.d/z10_spack_environment.sh /etc/profile.d/z10_spack_environment.sh

RUN apt-get -yqq update && apt-get -yqq upgrade \
 && apt-get -yqq install libgomp1 ssh \
 && rm -rf /var/lib/apt/lists/*


ENTRYPOINT ["/bin/bash", "--rcfile", "/etc/profile", "-l"]
