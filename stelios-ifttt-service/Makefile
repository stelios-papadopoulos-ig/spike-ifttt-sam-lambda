#https://gist.github.com/prwhite/8168133
# Add the following 'help' target to your Makefile
# And add help text after each target name starting with '\#\#'
 
help:           ## Show this help.
	@fgrep -h "##" $(MAKEFILE_LIST) | fgrep -v fgrep | sed -e 's/\\$$//' | sed -e 's/##//'
 
.PHONY: local-init local-run local-cleanup build

LOCAL_AWS_PROFILE=fake
MARKER_LOCAL=.marker_local_init

local-init:     ## (re)inititalise local resources (such as dynamo db). remember to local-cleanup
	docker-compose up -d
	echo using local aws profile '$(LOCAL_AWS_PROFILE)' If aws errors please set it up
	AWS_PROFILE=$(LOCAL_AWS_PROFILE) dynamodb/create-local-table.sh
	touch $(MARKER_LOCAL)

$(MARKER_LOCAL):
	$(MAKE) local-init

local-cleanup:  ## cleanup local resources (such as dynamo db)
	docker-compose down --volumes
	rm -f $(MARKER_LOCAL)

cleanup: ## cleanup everything
cleanup: local-cleanup
	rm -rf ./aws-sam package

local-run:      ## start api with local dynamo db
local-run: $(MARKER_LOCAL) package build
	sam local start-api --docker-network=local-sam

package: ## refresh pacakge dependencies
package: layer_package_requirements.txt
	rm -rf package
	pip install --target $@/python/lib/python3.8/site-packages/ -r $^
	

build: package
	sam build

deploy:
	test -n '$(API_KEY)' # API_KEY must be non empty
	test -n '$(AWS_PROFILE)' # AWS_PROFILE must be non empty
	AWS_PROFILE=$(AWS_PROFILE) sam deploy --parameter-overrides 'ApiKey=$(API_KEY)'
	