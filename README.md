# Playwright Test Case Script Generator

A SOLO Skill that automatically generates executable Playwright test scripts from structured test cases (Steps + Expected Result), with automatic Step alignment self-check.

## Overview

This skill transforms test cases into production-ready Playwright test scripts by:

1. **Analyzing** test case structure (Steps + Expected Result)
2. **Referencing** existing project scripts for style consistency
3. **Generating** executable `*.spec.ts` files with proper POM integration
4. **Self-checking** via step-by-step alignment verification

## Features

- Supports `@web`, `@Server`, and `@API` script types
- Reference-based generation ensures style consistency
- Automatic Step alignment verification
- POM method reuse and minimal增量 changes
- Block/Major/Minor issue classification

## Usage

### Input Requirements

1. **Test Case**: ID/Name, Precondition, Steps + Expected Result, Test data
2. **Runtime Environment**: Script type (`@web`/`@Server`/`@API`), locale
3. **Reference Script**: At least 1 similar `*.spec.ts` from the project

### Generation Flow

```
Step 0: Classify script type (@web/@Server/@API)
Step 1: Search for reference scripts in target directory
Step 2: Pair actions with assertions per test case step
Step 3: Add missing POM methods (minimal changes)
Step 4: Apply project template and style conventions
Step 5: Verify executability (no test.only/skip/commented assertions)
Step 6: Run self-check loop until all steps pass
```

## Output

- New `*.spec.ts` file (executable)
- Minimal POM/Helper incremental changes
- Step alignment self-check report

## Prerequisites

- Playwright test framework
- Existing POM/Helper structure in project
- Reference scripts for style consistency

## Related

- [testcase-step-alignment](https://github.com/ccbhwy/testcase-step-alignment) - Step alignment verification skill used for self-check