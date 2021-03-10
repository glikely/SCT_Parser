# SCT_Parser

This is an external Parser script for UEFI SCT. (WIP)

It's designed to read a `.ekl` results log from an UEFI SCT run, and a generated `.seq` from UEFI SCT configurator.

It will proceed to generate a Markdown file listing number of failures, passes, each test from the sequence file set that was Silently dropped, and a list of all failures and warnings.


## Usage
Usage to generate a "result md" is such. `python3 parser.py <log_file.ekl> <seq_file.seq>`
If you do no provided any command line arguments it will use `sample.ekl` and `sample.seq`.
The output filename can be specified with `--md <filename>`.

An online help is available with the `-h` option.

### Custom search
For a custom Key:value search, the next two arguments *MUST be included together.* The program will search and display files that met that constraint, without the crosscheck, and display the names, guid, and key:value to the command line. `python3 Parser.py <file.ekl> <file.seq> <search key> <search value>`

you can use the `test_dict` below to see available keys.

### Sorting data

It is possible to sort the tests data before output using
the `--sort <key1,key2,...>` option.
Sorting the test data helps when comparing results with diff.

Example command:

``` {.sh}
$ ./Parser.py --sort \
      'group,descr,set guid,test set,sub set,guid,name,log' ...
```

## Configuration file

It is possible to use a configuration file with command line option `--config
<filename>`.
This configuration file describes operations to perform on the tests results,
such as marking tests as false positives or waiving failures.

Example command for EBBR:

``` {.sh}
$ ./Parser.py --config EBBR.yaml /path/to/Summary.ekl EBBR.seq ...
```

You need to install the PyYAML module for this to work (see
<https://github.com/yaml/pyyaml>).

### Configuration file format

The configuration file is in YAML format (see <https://yaml.org>).
It contains a list of rules:

``` {.yaml}
- rule: name/description (optional)
  criteria:
    key1: value1
    key2: value2
    ...
  update:
    key3: value3
    key4: value4
    ...
- rule...
```

### Rule processing

The rules will be applied to each test one by one in the following manner:

* An attempt is made at matching all the keys/values of the rule's 'criteria'
  dict to the keys/values of the test dict. Matching test and criteria is done
  with a "relaxed" comparison (more below).
  - If there is no match, processing moves on to the next rule.
  - If there is a match:
    1. The test dict is updated with the 'update' dict of the rule.
    2. An 'Updated by' key is set in the test dict to the rule name.
    3. Finally, no more rule is applied to that test.

A test value and a criteria value match if the criteria value string is present
anywhere in the test value string.
For example, the test value "abcde" matches the criteria value "cd".

You can use `--debug` to see more details about which rules are applied to the
tests.

## Notes
### Known Issues:
* "comment" is currently not implemented, as formatting is not currently consistent, should reflect the comments from the test.
* some SCT tests have shared GUIDs,
* some lines in ekl file follow Different naming Conventions
* some tests in the sequence file are not strongly Associated with the test spec.

### Documentation

It is possible to convert this `README.md` into `README.pdf` with pandoc using
`make doc`. See `make help`.

### TODO:
* double check concatenation of all `.ekl` logs, preliminary tests show small Divergence between them and `summary.ekl` found in `Overall` folder. Cat.sh will generate this file.
* look into large number of dropped tests.


### db structure:
``` {.python}
tests = [
    test_dict,
    est_dict2...
]

test_dict = {
   "name": "some test",
   "result": "pass/fail",
   "group": "some group",
   "test set": "some set",
   "sub set": "some subset",
   "set guid": "XXXXXX",
   "guid": "XXXXXX",
   "comment": "some comment",
   "log": "full log output"
}


seqs = {
    <guid> : seq_dict
    <guid2> : seq_dict2...
}

seq_dict = {
                "name": "set name",
                "guid": "set guid",
                "Iteration": "some hex/num of how many times to run",
                "rev": "some hex/numb",
                "Order": "some hex/num"
}
```
