##############################################################################
# Copyright (c) 2016 Red Hat Inc.
# dradez@redhat.com
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Apache License, Version 2.0
# which accompanies this distribution, and is available at
# http://www.apache.org/licenses/LICENSE-2.0
##############################################################################

export USE_MASTER = ""
export RELEASE = "0"

export RPM_DIST = $(shell rpm -E %dist)
export RPMVERS = $(shell grep Version $(shell pwd)/rpm_specs/opnfv-apex.spec | head -n 1 | awk '{ print $$2 }')

export BUILD_ROOT = $(shell pwd)
export BUILD_DIR = $(shell dirname $$(pwd))/.build
export CACHE_DIR = $(shell dirname $$(pwd))/.cache
export RPM_DIR_ARGS = -D '_topdir $(BUILD_DIR)' -D '_builddir $(BUILD_DIR)' -D '_sourcedir $(BUILD_DIR)' -D '_rpmdir $(BUILD_DIR)' -D '_specdir $(BUILD_DIR)' -D '_srcrpmdir $(BUILD_DIR)'
export RPMCOM = $(BUILD_DIR)/noarch/python34-opnfv-apex-$(RPMVERS)-$(shell echo ${RELEASE} | tr -d '_-').noarch.rpm

export NETVPP_VERS := $(shell grep Version $(shell pwd)/rpm_specs/networking-vpp.spec | head -n 1 | awk '{ print $$2 }')
export NETVPP_REPO := "https://github.com/fepan/networking-vpp"
export NETVPP_BRANCH := "test-fdio-fix"
export NETVPP_COMMIT := $(shell git ls-remote $(NETVPP_REPO) $(NETVPP_BRANCH) | awk '{print substr($$1,1,7)}')

.PHONY: all
all: rpm

.PHONY: clean
clean:
	rm -rf $(BUILD_DIR)

.PHONY: clean-cache
clean-cache:
	rm -rf $(CACHE_DIR)

.PHONY: rpm-check
rpm-check: apex-rpm-check

.PHONY: rpm-clean
rpm-clean:
	rm -rf $(BUILD_DIR)/noarch
	rm -rf $(BUILD_DIR)/BUILDROOT

$(BUILD_DIR)/opnfv-apex.tar.gz:
	mkdir -p $(BUILD_DIR)
	pushd ../ && git archive --format=tar.gz --prefix=opnfv-apex-$(RPMVERS)/ HEAD > $(BUILD_DIR)/opnfv-apex.tar.gz

.PHONY: apex-rpm-check
apex-rpm-check: $(BUILD_DIR)/opnfv-apex.tar.gz
	rpmbuild --clean -bi -bl rpm_specs/opnfv-apex.spec $(RPM_DIR_ARGS) -D "_release $(shell echo $(RELEASE) | tr -d '_-')"

.PHONY: rpm
rpm: $(BUILD_DIR)/opnfv-apex.tar.gz $(RPMCOM)

$(RPMCOM):
	@echo "Building the Apex RPM"
	# build the Apex RPM
	rpmbuild --clean -ba rpm_specs/opnfv-apex.spec $(RPM_DIR_ARGS) -D "_release $(shell echo $(RELEASE) | tr -d '_-')"


##################
# NETWORKING-VPP #
##################

$(BUILD_DIR)/python-networking-vpp.tar.gz:
	@echo "Preparing the networking-vpp RPM prerequisites"
	git clone $(NETVPP_REPO) -b $(NETVPP_BRANCH) $(BUILD_DIR)/python-networking-vpp-$(NETVPP_VERS)
	tar czf $(BUILD_DIR)/python-networking-vpp.tar.gz -C $(BUILD_DIR) python-networking-vpp-$(NETVPP_VERS)

.PHONY: networking-vpp-rpm
networking-vpp-rpm: $(BUILD_DIR)/noarch/python-networking-vpp-*.noarch.rpm

$(BUILD_DIR)/noarch/python-networking-vpp-*.noarch.rpm: $(BUILD_DIR)/python-networking-vpp.tar.gz
	@echo "Building the Networking VPP RPM"
	rpmbuild --clean -ba --target noarch rpm_specs/networking-vpp.spec $(RPM_DIR_ARGS) -D 'git .git$(NETVPP_COMMIT)'

##################
#  PYTHON TESTS  #
##################

.PHONY: python-tests
python-tests:
	tox -e py35

#######################
#  PYTHON PEP8 CHECK  #
#######################

.PHONY: python-pep8-check
python-pep8-check:
	tox -e pep8

#############
#  YAMLLINT #
#############

.PHONY: yamllint
yamllint:
	@echo "Running yamllint against all .yaml files"
	cd ../ && yamllint $(shell cd ../ && git ls-tree -r HEAD --name-only | grep 'yaml$$')
#############
#  RPMLINT  #
#############

.PHONY: rpmlint
rpmlint:
	@echo "Running rpmlint against all RPM spec files"
	rpmlint rpm_specs/*.spec

