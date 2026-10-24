# php-src playground

Step through PHP's C source (`php-src`) while it runs `test.php`.

## Setup

Clone php-src into this folder (it's gitignored):

```sh
git clone https://github.com/php/php-src.git
```

Then build it:

```sh
sudo apt install -y build-essential autoconf bison re2c gdb
cd php-src
./buildconf && ./configure --disable-all --enable-debug && make -j"$(nproc)"
```

F5 in VS Code reruns `make` first, so C edits are picked up. More detail: `php-src/README.md` and `php-src/docs/source/introduction/ides/visual-studio-code.rst`.

## Debug

Install the C/C++ extension, set a breakpoint, press F5.

## Debugging ZEND_HASH_INDEX_FIND

Breakpoints inside a macro's `#define` never fire. Break where it's used instead: `$array[2]` runs it at `php-src/Zend/zend_execute.c:2845` (in `zend_fetch_dimension_address_inner`):

```c
ZEND_HASH_INDEX_FIND(ht, hval, retval, num_undef);
```

After that line runs, watch `(char*)retval->value.str->val` to see the string (`"foo3"`).

## Debugging _zend_hash_index_add_or_update_i

Writes go through this function (`php-src/Zend/zend_hash.c:1271`). Unlike the macro, it's a real function, so set a function breakpoint on `_zend_hash_index_add_or_update_i` (or a line breakpoint at `zend_hash.c:1278`). With `test.php` it stops twice:

| PHP | Called from | `h` | `flag` |
|---|---|---|---|
| `$array[] = 'foo4'` | `zend_hash_next_index_insert` | `3` | `18` (`HASH_ADD \| HASH_ADD_NEXT`) |
| `$array[4] = 'foo5'` | `zend_hash_index_lookup` | `4` | `32` (`HASH_LOOKUP`) |

On the first stop, watch `(char*)pData->value.str->val` to see `"foo4"`. On the second, `pData` is `NULL`: the function only creates the empty slot, and the value is written into it afterwards.
