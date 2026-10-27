# Add your test source files here:
SOURCES = $(sort $(wildcard *.c))
CILS = $(sort $(wildcard *.cil))

###########################################################################

EXECUTABLE = libsemanage-tests
USER_JOIN_TEST = test-user-join-transaction
CFLAGS += -g -O0 -Wall -W -Wundef -Wmissing-noreturn -Wmissing-format-attribute
override CFLAGS += -I../src -I../include
override LDLIBS += -lcunit -lbz2 -laudit -lselinux -lsepol

OBJECTS = $(SOURCES:.c=.o)
POLICIES = $(CILS:.cil=.policy)

all: $(EXECUTABLE) $(POLICIES) test-config-empty $(USER_JOIN_TEST)

test-config-empty: regression/test_config_empty.c ../src/libsemanage.a
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^ -Wl,--wrap=strdup $(LDLIBS)
$(USER_JOIN_TEST): regression/test_user_join_transaction.c ../src/libsemanage.a
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^ $(LDLIBS)

$(EXECUTABLE): $(OBJECTS) ../src/libsemanage.a
	$(CC) $(LDFLAGS) -o $@ $^ $(LDLIBS)

%.policy: %.cil
	../../secilc/secilc $*.cil -o $*.policy -f /dev/null

clean distclean:
	rm -rf $(OBJECTS) $(POLICIES) $(EXECUTABLE) test-config-empty $(USER_JOIN_TEST)

test: all
	./test-config-empty
	./$(EXECUTABLE)
	./$(USER_JOIN_TEST)

# Requires SWIG, Python development headers and the built shared library.
PYTHON ?= python3
.PHONY: test-python-module-buffer
test-python-module-buffer:
	$(PYTHON) regression/test_module_buffer.py
