HOST_SYSTEM = $(shell uname | cut -f 1 -d_)
SYSTEM ?= $(HOST_SYSTEM)
CXX = g++
CPPFLAGS += `pkg-config --cflags protobuf grpc`
CXXFLAGS += -std=c++17
ifeq ($(SYSTEM),Darwin)
LDFLAGS += -L/usr/local/lib `pkg-config --libs protobuf grpc++`\
					 -pthread\
           -lgrpc++_reflection\
           -ldl
else
LDFLAGS += -L/usr/local/lib `pkg-config --libs protobuf grpc++`\
           -pthread\
           -Wl,--no-as-needed -lgrpc++_reflection -Wl,--as-needed\
           -ldl
endif
PROTOC = protoc
GRPC_CPP_PLUGIN = grpc_cpp_plugin
GRPC_CPP_PLUGIN_PATH ?= `which $(GRPC_CPP_PLUGIN)`

GOOGLE_TEST_LIB = gtest
GOOGLE_TEST_INCLUDE = /usr/local/include
G++_FLAGS = -c -Wall -I $(GOOGLE_TEST_INCLUDE)
TEST_LD_FLAGS = -L /usr/local/lib -l $(GOOGLE_TEST_LIB) -l pthread



PROTOS_PATH = ../../protos

vpath %.proto $(PROTOS_PATH)

all: kvstore_client kvstore_server 

kvstore_client: kvstore.pb.o kvstore.grpc.pb.o kvstore_client.o
	$(CXX) $^ $(LDFLAGS) -o $@

kvstore_server: kvmap.o kvstore.pb.o kvstore.grpc.pb.o kvstore_server.o 
	$(CXX) $^ $(LDFLAGS) -o $@

kvmap_test: kvmap.o kvmap_test.o 
	$(CXX) -o $@ $^ $(TEST_LD_FLAGS)

.PRECIOUS: %.grpc.pb.cc
%.grpc.pb.cc: %.proto
	$(PROTOC) -I $(PROTOS_PATH) --grpc_out=. --plugin=protoc-gen-grpc=$(GRPC_CPP_PLUGIN_PATH) $<

.PRECIOUS: %.pb.cc
%.pb.cc: %.proto
	$(PROTOC) -I $(PROTOS_PATH) --cpp_out=. $<

clean:
	rm -f *.o *.pb.cc *.pb.h kvstore_client kvstore_server