# Add your test source files here:
SOURCES = $(sort $(wildcard *.c))
CILS = $(sort $(wildcard *.cil))
###########################################################################
EXECUTABLE = libsemanage-tests
USER_JOIN_TEST = test-user-join-transaction
CFLAGS += -g -O0 -Wall -W -Wundef -Wmissing-noreturn -Wmissing-format-attribute
override CFLAGS += -I../src -I../include
override LDLIBS += -lcunit -lbz2 -laudit -lselinux -lsepol
ERROR_TESTS = test-parse-read-error
ERROR_TESTS = test-read-lock-error test-parse-read-error

OBJECTS = $(SOURCES:.c=.o)

POLICIES = $(CILS:.cil=.policy)

all: $(EXECUTABLE) $(POLICIES) test-config-empty $(USER_JOIN_TEST) $(ERROR_TESTS)

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
	rm -rf $(OBJECTS) $(POLICIES) $(EXECUTABLE) $(USER_JOIN_TEST) $(ERROR_TESTS)

test: all
	./test-config-empty
	./$(EXECUTABLE)
	./$(USER_JOIN_TEST)
	./test-parse-read-error
	./test-read-lock-error

test-parse-read-error: regression/test_parse_read_error.c ../src/parse_utils.c
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $<

test-read-lock-error: regression/test_read_lock_error.c ../src/database.c
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $<

# Requires SWIG, Python development headers and the built shared library.
PYTHON ?= python3
.PHONY: test-python-module-buffer
test-python-module-buffer:
	$(PYTHON) regression/test_module_buffer.py
