import * as fs from 'fs'
import * as path from 'path'

import * as url from 'url' // failure
~~~~~~~~~~~~~~~~~~~~~~~~~~          [import-group-same]

const v = 'a' // failure
~~~~~~~~~~~~~                       [not-group]

import * as Typescript from 'typescript'
import * as Lint from 'tslint'
import { func } from '../lib' // failure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~        [import-group-diff]

import * as http from 'http' // failure
~~~~~~~~~~~~~~~~~~~~~~~~~~~           [import-group-ordered]

[import-group-same]:Unexpected empty line within the same import group.
[import-group-diff]:Expecting an empty line between different import groups.
[import-group-ordered]:Import groups must be sorted according to given order.
[not-group]:Imports must be grouped.'
