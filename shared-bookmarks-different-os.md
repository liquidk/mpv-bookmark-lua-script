If you have several OS installed on your system and you go back and forth between them; you probably want to use a shared `bookmarks.json` and put this file on a partition that is mounted on all your OSs. Then you need to modify two functions in the script:

* Firstly `getConfigFile()` and change it such that it points to a static file like this:
```
function getConfigFile()
  return platform_independent("/shared/mpv/bookmarks.json")
end
```

* Secondly implement `platform_independent()` function such that it takes care of platform-specific path-prefix; here's a simple mechanism that removes a prefix from the set, then tries all prefixes to see which works:

```
function platform_independent(filepath)
  function map(func, array)
    local new_array = {}
    for i,v in ipairs(array) do
      new_array[i] = func(v)
    end
    return new_array
  end
  function remove_prefixes(prefixes, path)
    new_path = path
    for _,p in ipairs(prefixes) do
      new_path = new_path:gsub("^(" .. p .. ")", "")
    end
    return new_path
  end
  function try_other_prefixes(prefixes, path)
    tail = remove_prefixes(prefixes, path)  
    for _, p in ipairs(prefixes) do
      if file_exists(p .. tail) then
        return p .. tail
      end
    end
    return path
  end
  if filepath == nil or filepath == "" then
    return filepath
  end
  -- NOW INSTRUCT WHICH GROUP OF PREFIXES POINTING TO THE SAME PATH
  filepath = try_other_prefixes({ '/Volumes/Archive/', 'd:/', 'D:/', '/d/' }, filepath)
  filepath = try_other_prefixes({ 'E:/home/nima/', 'e:/home/nima/', '/home/nima/' }, filepath)  
  return filepath
end
```