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

  Stack trace:

  11  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/clikit/console_application.py:131 in run
       129│             parsed_args = resolved_command.args
       130│ 
     → 131│             status_code = command.handle(parsed_args, io)
       132│         except KeyboardInterrupt:
       133│             status_code = 1

  10  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/clikit/api/command/command.py:120 in handle
       118│     def handle(self, args, io):  # type: (Args, IO) -> int
       119│         try:
     → 120│             status_code = self._do_handle(args, io)
       121│         except KeyboardInterrupt:
       122│             if io.is_debug():

   9  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/clikit/api/command/command.py:163 in _do_handle
       161│         if self._dispatcher and self._dispatcher.has_listeners(PRE_HANDLE):
       162│             event = PreHandleEvent(args, io, self)
     → 163│             self._dispatcher.dispatch(PRE_HANDLE, event)
       164│ 
       165│             if event.is_handled():

   8  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/clikit/api/event/event_dispatcher.py:22 in dispatch
        20│ 
        21│         if listeners:
     →  22│             self._do_dispatch(listeners, event_name, event)
        23│ 
        24│         return event

   7  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/clikit/api/event/event_dispatcher.py:89 in _do_dispatch
        87│                 break
        88│ 
     →  89│             listener(event, event_name, self)
        90│ 
        91│     def _sort_listeners(self, event_name):  # type: (str) -> None

   6  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/poetry/console/config/application_config.py:116 in set_env
       114│ 
       115│         io = event.io
     → 116│         poetry = command.poetry
       117│ 
       118│         env_manager = EnvManager(poetry)

   5  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/poetry/console/commands/command.py:10 in poetry
        8│     @property
        9│     def poetry(self):
     → 10│         return self.application.poetry
       11│ 
       12│     def reset_poetry(self):  # type: () -> None

   4  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/poetry/console/application.py:69 in poetry
        67│             return self._poetry
        68│ 
     →  69│         self._poetry = Factory().create_poetry(Path.cwd())
        70│ 
        71│         return self._poetry

   3  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/poetry/factory.py:33 in create_poetry
        31│             io = NullIO()
        32│ 
     →  33│         base_poetry = super(Factory, self).create_poetry(cwd)
        34│ 
        35│         locker = Locker(

   2  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/poetry/core/factory.py:93 in create_poetry
        91│ 
        92│                 package.add_dependency(
     →  93│                     self.create_dependency(name, constraint, root_dir=package.root_dir)
        94│                 )
        95│ 

   1  ~/Library/Application Support/pypoetry/venv/lib/python3.7/site-packages/poetry/core/factory.py:251 in create_dependency
       249│                         base=root_dir,
       250│                         develop=constraint.get("develop", False),
     → 251│                         extras=constraint.get("extras", []),
       252│                     )
       253│             elif "url" in constraint:

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