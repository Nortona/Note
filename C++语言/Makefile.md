## version 1

```makefile
hello: main.cpp printhello.cpp factorial.cpp
	g++ -o hello main.cpp printhello.cpp factorial.cpp 
```
hello为生成目标
main.cpp printhello.cpp factorial.cpp为依赖,为目标编译文件，是我们的依赖文件
采用的命令为`	g++ -o hello main.cpp printhello.cpp factorial.cpp `


## version 2

```makefile
CXX = g++
TARGET = hello
OBJ = main.o printhello.o factorical.o

// TARGET 变量依赖于OBJ，若OBJ有文件更新，执行下方命令
$(TARGET) : $(OBJ)
	$(CXX) -o $(TARGET) $(OBJ)

main.o: main.cpp
	$(CXX) -c main.cpp

printhello.o: printhello.cpp
	$(CXX) -c printhello.cpp

factorical.o: factorical.cpp
	$(CXX) -c factorical.cpp
```

## version 3
```makefile
CXX = g++
TARGET = hello
OBJ = main.o printhello.o factorical.o

CXXFLAGS = -c -Wall

$(TARGET) : $(OBJ)
	$(CXX) -o $@ $^

$.o : %.cpp
	$(CXX) $(CXXFLAGS) $< -o $@

.PHONY : clean // 防止有文件为clean
clean:
	rm -f *.o $(TARGET)

```