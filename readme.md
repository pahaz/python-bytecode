PROOFS
======

## compile code ##

    >>> code_str = """
    ... print "Hello, world"
    ... """
    >>> code_obj = compile(code_str, '<string>', 'exec')
    >>> code_obj
    <code object <module> at 0x1054c74b0, file "<string>", line 2>

The first argument to compile() is the string of Python code to be compiled, which should be obvious. The second defines the “filename” of the piece of code (here, as is conventional, we use <string> to indicate code attained from the interactive shell). The third is the type of compilation, which most often will be exec as you see here. The other choices for mode are eval, which is used for strings containing only a single expression, or single, in which the generated code object is expected to contain a single statement, whose return value is printed if it is not None (like in the interactive shell).

From: http://late.am/post/2012/03/26/exploring-python-code-objects

## decompillers ##

 * [UnPyc](http://sourceforge.net/projects/unpyc/)
 * [uncompyle2](https://github.com/wibiti/uncompyle2)

## structure of .pyc files ##

showStructurePyc.py

    import dis, marshal, struct, sys, time, types

    def show_file(fname):
        f = open(fname, "rb")
        magic = f.read(4)
        moddate = f.read(4)
        modtime = time.asctime(time.localtime(struct.unpack('L', moddate)[0]))
        print "magic %s" % (magic.encode('hex'))
        print "moddate %s (%s)" % (moddate.encode('hex'), modtime)
        code = marshal.load(f)
        show_code(code)

    def show_code(code, indent=''):
        print "%scode" % indent
        indent += '   '
        print "%sargcount %d" % (indent, code.co_argcount)
        print "%snlocals %d" % (indent, code.co_nlocals)
        print "%sstacksize %d" % (indent, code.co_stacksize)
        print "%sflags %04x" % (indent, code.co_flags)
        show_hex("code", code.co_code, indent=indent)
        dis.disassemble(code)
        print "%sconsts" % indent
        for const in code.co_consts:
            if type(const) == types.CodeType:
                show_code(const, indent+'   ')
            else:
                print "   %s%r" % (indent, const)
        print "%snames %r" % (indent, code.co_names)
        print "%svarnames %r" % (indent, code.co_varnames)
        print "%sfreevars %r" % (indent, code.co_freevars)
        print "%scellvars %r" % (indent, code.co_cellvars)
        print "%sfilename %r" % (indent, code.co_filename)
        print "%sname %r" % (indent, code.co_name)
        print "%sfirstlineno %d" % (indent, code.co_firstlineno)
        show_hex("lnotab", code.co_lnotab, indent=indent)

    def show_hex(label, h, indent):
        h = h.encode('hex')
        if len(h) < 60:
            print "%s%s %s" % (indent, label, h)
        else:
            print "%s%s" % (indent, label)
            for i in range(0, len(h), 60):
                print "%s   %s" % (indent, h[i:i+60])

    show_file(sys.argv[1])

Running this on the test.pyc from an ultra-simple Python file:

    a, b = 1, 0
    if a or b:
        print "Hello", a

Run showStructurePyc.py:

    $python showStructurePyc.py test.pyc
    magic b3f20d0a
    moddate 8a9efc47 (Wed Apr 09 06:46:34 2008)
    code
       argcount 0
       nlocals 0
       stacksize 2
       flags 0040
       code
          6404005c02005a00005a0100650000700700016501006f0d000164020047
          65000047486e01000164030053
      1           0 LOAD_CONST               4 ((1, 0))
                  3 UNPACK_SEQUENCE          2
                  6 STORE_NAME               0 (a)
                  9 STORE_NAME               1 (b)

      2          12 LOAD_NAME                0 (a)
                 15 JUMP_IF_TRUE             7 (to 25)
                 18 POP_TOP
                 19 LOAD_NAME                1 (b)
                 22 JUMP_IF_FALSE           13 (to 38)
            >>   25 POP_TOP

      3          26 LOAD_CONST               2 ('Hello')
                 29 PRINT_ITEM
                 30 LOAD_NAME                0 (a)
                 33 PRINT_ITEM
                 34 PRINT_NEWLINE
                 35 JUMP_FORWARD             1 (to 39)
            >>   38 POP_TOP
            >>   39 LOAD_CONST               3 (None)
                 42 RETURN_VALUE
       consts
          1
          0
          'Hello'
          None
          (1, 0)
       names ('a', 'b')
       varnames ()
       freevars ()
       cellvars ()
       filename 'C:\\ned\\sample.py'
       name '<module>'
       firstlineno 1
       lnotab 0c010e01

From: http://nedbatchelder.com/blog/200804/the_structure_of_pyc_files.html

## patch and create byte code ##

 * [byteplay](http://wiki.python.org/moin/ByteplayDoc)
 * [BytecodeAssembler](http://peak.telecommunity.com/DevCenter/BytecodeAssembler)

## some links ##

marshal works^
 * https://bitbucket.org/python_mirrors/sandbox-jython27/src/4bfd719d27fe/Lib/marshal.py
 * http://svn.python.org/view/python/trunk/Python/marshal.c?view=markup

## russian links ##

 * http://old.tltsu.ru/archive/doc/programming/python/lib_py/lib152/module-dis.html
 * [Самая короткая запись асинхронных вызовов в tornado или патчим байткод в декораторе](http://habrahabr.ru/post/153595/)
 * http://python-lab.blogspot.ru/2012/05/exploring-python-code-objects.html
 * [Модифицикация байт-кода функции в Python](http://habrahabr.ru/post/140356/)

Python Bytecode - Behind the Scenes
===================================

Tema: Seguridad/Ingeniería inversa sobre Python

Licencia: CC BY NC SA

* Python Bytecode: de py a pyc
    - Compilar código
    - Estructura de los archivos .pyc

* Presentación de los módulos marshall[1] y dis[2]
    - Obtener code objects
    - Ejecutar code objects (eval, exec)
    - Desensamblar code objects

* Experimentos:
    - byteplay[3]: manipulando code objects
    - uncompyle[4]: de pyc a py

* Caso de estudio: haciendo un poco de ingeniería inversa sobre un "crackme"[5] simple
    - Extraer el code object de un exe compilado con py2exe
    - Desensamblar y entender el programa
    - Generar el serial válido

* Algunas protecciones existentes

[1] http://docs.python.org/library/dis.html

[2] http://docs.python.org/library/marshal.html

[3] http://code.google.com/p/byteplay/

[4] https://github.com/matiasb/uncompyle

[5] http://en.wikipedia.org/wiki/Crackme
