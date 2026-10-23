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
ERROR_TESTS = test-read-lock-error test-user-delete-error test-parse-read-error
LIFECYCLE_TEST = test-handle-lifecycle
SERIAL_TESTS = test-cache-serial-file test-cache-serial-join test-cache-serial-active test-cache-serial-policydb
ROLLBACK_TEST = test-final-rollback
RESTART_TEST = test-transaction-restart

OBJECTS = $(SOURCES:.c=.o)

POLICIES = $(CILS:.cil=.policy)

all: $(EXECUTABLE) $(POLICIES) test-config-empty $(USER_JOIN_TEST) $(ERROR_TESTS) $(LIFECYCLE_TEST) $(SERIAL_TESTS) $(ROLLBACK_TEST) $(RESTART_TEST)

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
	rm -rf $(OBJECTS) $(POLICIES) $(EXECUTABLE) $(USER_JOIN_TEST) $(LIFECYCLE_TEST) $(ERROR_TESTS)
	rm -rf $(OBJECTS) $(POLICIES) $(EXECUTABLE) $(USER_JOIN_TEST) $(LIFECYCLE_TEST) $(ERROR_TESTS) $(SERIAL_TESTS)
	rm -rf $(OBJECTS) $(POLICIES) $(EXECUTABLE) $(USER_JOIN_TEST) $(LIFECYCLE_TEST) $(ERROR_TESTS) $(SERIAL_TESTS) $(ROLLBACK_TEST)
	rm -rf $(OBJECTS) $(POLICIES) $(EXECUTABLE) $(USER_JOIN_TEST) $(LIFECYCLE_TEST) $(ERROR_TESTS) $(SERIAL_TESTS) $(ROLLBACK_TEST) $(RESTART_TEST)

test: all test-ibpkey-validation
	./test-config-empty
	./$(EXECUTABLE)
	./$(USER_JOIN_TEST)
	./test-parse-read-error
	./test-read-lock-error
	./test-user-delete-error
	./$(LIFECYCLE_TEST)
	@for test in $(SERIAL_TESTS); do ./$$test || exit 1; done
	./$(ROLLBACK_TEST)
	./$(RESTART_TEST)

test-parse-read-error: regression/test_parse_read_error.c ../src/parse_utils.c
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $<

test-read-lock-error: regression/test_read_lock_error.c ../src/database.c
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $<

test-user-delete-error: regression/test_user_delete_error.c ../src/users_local.c
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $<

$(LIFECYCLE_TEST): regression/test_handle_lifecycle.c ../src/libsemanage.a
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^ $(LDLIBS)

test-cache-serial-file: SERIAL_BACKEND = FILE

test-cache-serial-join: SERIAL_BACKEND = JOIN

test-cache-serial-active: SERIAL_BACKEND = ACTIVE

test-cache-serial-policydb: SERIAL_BACKEND = POLICYDB

$(SERIAL_TESTS): regression/test_cache_serial_error.c ../src/database_llist.c \
		../src/database_llist.h ../src/database_file.c \
		../src/database_join.c ../src/database_activedb.c \
		../src/database_policydb.c
	$(CC) $(CFLAGS) $(LDFLAGS) -DTEST_$(SERIAL_BACKEND) -o $@ $< ../src/database_llist.c -lsepol

$(ROLLBACK_TEST): regression/test_final_rollback.c ../src/libsemanage.a
	$(CC) $(CFLAGS) $(LDFLAGS) -Wl,--wrap=fsync -Wl,--wrap=rename -Wl,--wrap=link -Wl,--wrap=unlinkat -Wl,--wrap=glob -o $@ $^ $(LDLIBS)

$(RESTART_TEST): regression/test_transaction_restart.c ../src/libsemanage.a
	$(CC) $(CFLAGS) $(LDFLAGS) -Wl,--wrap=scandir -o $@ $^ $(LDLIBS)

test-ibpkey-validation:
	sh ibpkey-validation/run.sh

.PHONY: test-ibpkey-validation

# Requires SWIG, Python development headers and the built shared library.
PYTHON ?= python3
.PHONY: test-python-module-buffer
test-python-module-buffer:
	$(PYTHON) regression/test_module_buffer.py
