HOST_SYSTEM = $(shell uname | cut -f 1 -d_)
SYSTEM ?= $(HOST_SYSTEM)
CXX = g++
CPPFLAGS += `pkg-config --cflags protobuf grpc`
CXXFLAGS += -std=c++17
CXXFLAGS += -g
ifeq ($(SYSTEM),Darwin)
LDFLAGS += -L/usr/local/lib `pkg-config --libs protobuf grpc++`\
					 -pthread\
					 -lglog\
           -lgrpc++_reflection\
           -ldl
else
LDFLAGS += -L/usr/local/lib `pkg-config --libs protobuf grpc++`\
           -lglog\
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

server_objects = ../kvstore/kvstore.pb.o\
				 ../kvstore/kvstore.grpc.pb.o\
				 ../kvstore/kvstore_client.o\
				 func.pb.o\
				 func.grpc.pb.o\
				 func_server.o\
				 warble.pb.o\
				 warble.grpc.pb.o\
				 func_infra.o\
				 warble_func.o\

				
test_objects =   ../kvstore/kvstore.pb.o\
				 ../kvstore/kvstore.grpc.pb.o\
				 ../kvstore/kvstore_local.o\
				 ../kvstore/kvmap.o\
				 warble.pb.o\
				 warble.grpc.pb.o\
				 warble_func.o\
				 warble_func_test.o\
				 

vpath %.proto $(PROTOS_PATH)

all: warble.pb.cc warble.grpc.pb.cc func_server warble_func_test

warble.grpc.pb.cc:
	$(PROTOC) -I $(PROTOS_PATH) --grpc_out=. --plugin=protoc-gen-grpc=$(GRPC_CPP_PLUGIN_PATH) warble.proto

warble.pb.cc:
	$(PROTOC) -I $(PROTOS_PATH) --cpp_out=. warble.proto

func_server: $(server_objects)
	$(CXX) $^ $(LDFLAGS) -o $@

func_infra_test: func_infra.o func_infra_test.o 
	$(CXX) -o $@ $^ $(TEST_LD_FLAGS)

warble_func_test: $(test_objects)
	$(CXX) $(CXXFLAGS) -I . -o $@ $^ $(TEST_LD_FLAGS) $(LDFLAGS)

.PRECIOUS: %.grpc.pb.cc
%.grpc.pb.cc: %.proto
	$(PROTOC) -I $(PROTOS_PATH) --grpc_out=. --plugin=protoc-gen-grpc=$(GRPC_CPP_PLUGIN_PATH) $<

.PRECIOUS: %.pb.cc
%.pb.cc: %.proto
	$(PROTOC) -I $(PROTOS_PATH) --cpp_out=. $<

clean:
	rm -f *.o *.pb.cc *.pb.h func_client func_server warble_func_test
