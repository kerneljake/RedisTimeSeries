#set environment variable RM_INCLUDE_DIR to the location of redismodule.h
ifndef RM_INCLUDE_DIR
	RM_INCLUDE_DIR=../RedisModulesSDK
endif

ifndef RMUTIL_LIBDIR
	RMUTIL_LIBDIR=../RedisModulesSDK/rmutil
endif

# find the OS
uname_S := $(shell sh -c 'uname -s 2>/dev/null || echo not')

# Compile flags for linux / osx
ifeq ($(uname_S),Linux)
	SHOBJ_CFLAGS ?=  -fno-common -g -ggdb
	SHOBJ_LDFLAGS ?= -shared -Bsymbolic
else
	SHOBJ_CFLAGS ?= -dynamic -fno-common -g -ggdb
	SHOBJ_LDFLAGS ?= -bundle -undefined dynamic_lookup
endif
CFLAGS = -I$(RM_INCLUDE_DIR) -Wall -O3 -g -fPIC -lc -lm -std=gnu99  -DREDIS_MODULE_TARGET -DREDISMODULE_EXPERIMENTAL_API

all: rmutil redistimeseries.so

rmutil:
	$(MAKE) -C $(RMUTIL_LIBDIR)

redistimeseries.so: rmutil module.o tsdb.o compaction.o rdb.o chunk.o parse_policies.o config.o indexer.o endianconv.o
	$(LD) -o $@ module.o tsdb.o rdb.o compaction.o chunk.o parse_policies.o config.o indexer.o $(SHOBJ_LDFLAGS) $(LIBS) -L$(RMUTIL_LIBDIR) -lrmutil -lc

clean:
	rm -rf *.xo *.so *.o ./tests_runner

tests: redistimeseries.so
	python2 -m pytest ${EXTRA_ARGS} -svv .

run: redistimeseries.so
	redis-server --loadmodule `pwd`/redistimeseries.so COMPACTION_POLICY "" RETNTION_POLICY 3600 MAX_SAMPLE_PER_CHUNK 1024 --dir /tmp

unittests_runner: redistimeseries.so tests.o
	$(CC) *.o -o unittests_runner $(LIBS) -L$(RMUTIL_LIBDIR) -lrmutil -lc -lm

unittests: unittests_runner
	./unittests_runner

docker:
	cd .. && docker build -t redis-tsdb .

package: redistimeseries.so
	mkdir -p ../build
	ramp pack -m "`pwd`/../ramp.yml" -v -o "../build/redistimeseries.{os}-{architecture}.latest.zip" "`pwd`/redistimeseries.so"

.PHONY: package tests unittests clean all
