# 순서
1. `CMakeLists.txt` 에서 프로젝트명 정하기
2. `cmake/ProjectConfig.cmake.in` 파일명 바꾸고 프로젝트명 정하기
3. 다음 실행하기
```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release # Linux
cmake -S . -B build -G "MinGW Makefiles" # Windows
cmake --build build --config Release
```