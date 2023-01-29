CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c11 -ggdb
TST_LIBS := -lcheck -lm -lpthread
OS := $(shell uname -s)

ifeq ($(OS),Linux)
	CFLAGS+=-D LINUX
	TST_LIBS+= -lcheck_pic -lrt -lsubunit
else
	ifeq ($(OS),Darwin)
		CFLAGS+=-D MAC
	endif
endif

all : test gcov_report

test: s21_string.a s21_test.o
	$(CC) $(CFLAGS) s21_test.o s21_string.a $(TST_LIBS) -o test
	@./test

re : clean test gcov_report

s21_test.o: s21_string_test.c
	$(CC) $(CFLAGS) -c s21_string_test.c -o s21_test.o

s21_string.o: s21_string.c
	$(CC) $(CFLAGS) -c s21_string.c -o s21_string.o

s21_string.a: s21_string.o s21_sscanf.o
	@ar rcs s21_string.a s21_string.o s21_sscanf.o
	ranlib s21_string.a

s21_sscanf.o: s21_sscanf.c
	$(CC) $(CFLAGS) -c s21_sscanf.c -o s21_sscanf.o

gcov_report: gcov_test.info
	@genhtml -o report/ gcov_test.info
	@open report/index.html

gcov_test.info: gcov_test gcov_test.gcda
	@lcov -t "gcov_test" -o gcov_test.info --no-external -c -d .

gcov_test: s21_string_test.c s21_string.a 
	$(CC) $(CFLAGS) --coverage s21_string_test.c s21_string.c s21_string.a $(TST_LIBS) -o gcov_test

gcov_test.gcda: gcov_test
	@chmod +x *
	@./gcov_test

clean : cleanbuild cleanlibs
cleanbuild :
	@rm -rf build/* report/*
cleanlibs :
	@rm -rf *.o *.a test gcov_test *.gcno *.gcda *.info

leak:
	CK_FORK=no leaks -atExit -- ./test
	CK_FORK=no leaks -atExit -- ./gcov_test

cpp_lint_h:
	python3 ../materials/linters/cpplint.py *.h

cpp_lint_c:
	python3 ../materials/linters/cpplint.py *.c

cppcheck:
	cppcheck s21_string.c s21_string_test.c

valgrind:
	CK_FORK=no valgrind --leak-check=full --track-origins=yes --show-leak-kinds=all -s ./test
	CK_FORK=no valgrind --leak-check=full --track-origins=yes --show-leak-kinds=all -s ./gcov_test
	# valgrind --leak-check=full --track-origins=yes --trace-children=yes -s ./test                  для линукса
    # valgrind --leak-check=full --track-origins=yes --trace-children=yes -s ./gcov_test
