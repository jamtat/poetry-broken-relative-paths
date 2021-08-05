# Broken Relative Paths when package in subtree of another package


Relative path imports fail when there is a dependency tree such that:
- `application -> otherlib -> baselib`
- `otherlib` and `baselib` share a filepath with pyproject.toml such that if pyproject.toml is at `<path>/pyproject.toml`, other libs are at `<path>/<any descendant path>/{otherlib,baselib}`

## Steps to reproduce
- Have poetry 1.1.17
- Latest pip
- Run:
```
pushd common/baselib; poetry install; popd
pushd common/otherlib; poetry install; popd
poetry install
```

## Result

```
  ValueError

  Directory ../common/otherlib does not exist

  at ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/poetry/core/packages/directory_dependency.py:41 in __init__
       37│         self._develop = develop
       38│         self._supports_poetry = False
       39│ 
       40│         if not self._full_path.exists():
    →  41│             raise ValueError("Directory {} does not exist".format(self._path))
       42│ 
       43│         if self._full_path.is_file():
       44│             raise ValueError("{} is a file, expected a directory".format(self._path))
       45│ 
```

## Expected result

The relative path is corrected to be relative to the root pyproject.toml and the package is installed correctly

## Directory Structure:
```
|- pyproject.toml
|- application/
   |- __init__.py
|- common
   |- baselib/
      |- pyproject.toml
      |- baselib/
         |- __init__.py
   |- otherlib/
      |- pyproject.toml
      |- otherlib/
         |- __init__.py
```