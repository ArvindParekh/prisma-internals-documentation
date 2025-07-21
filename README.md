# @prisma/internals Package Documentation

## Overview

`@prisma/internals` is an internal package used by Prisma for various utility functions and tools. It was previously known as `@prisma/sdk` (renamed in Prisma 4.0.0) and is primarily intended for Prisma's internal use. While not officially documented or supported, it has become widely used in the community for building Prisma generators and other tools.

**⚠️ Important Warnings:**
- This package does not follow semantic versioning
- Breaking changes can occur without warning
- Not recommended for production use
- No official support or documentation from Prisma
- API may change at any time

## Installation

```bash
npm install @prisma/internals
```

## Package Information

- **License**: Apache-2.0
- **Repository**: https://github.com/prisma/prisma
- **Homepage**: https://www.prisma.io
- **Description**: This package is intended for Prisma's internal use
- **Main**: dist/index.js
- **Types**: dist/index.d.ts

## Core Functions and APIs

### 1. getDMMF (Get Data Model Meta Format)

The most commonly used function from this package. It parses a Prisma schema and returns the DMMF (Data Model Meta Format), which is an AST (Abstract Syntax Tree) representation of your Prisma schema.

```typescript
import { getDMMF } from '@prisma/internals'

// Basic usage with file path
const dmmf = await getDMMF({
  datamodelPath: './prisma/schema.prisma'
})

// With datamodel string
const dmmf = await getDMMF({
  datamodel: `
    model User {
      id    Int    @id
      email String @unique
      name  String?
      posts Post[]
    }
    
    model Post {
      id       Int    @id
      title    String
      author   User   @relation(fields: [authorId], references: [id])
      authorId Int
    }
  `
})

// With additional options
const dmmf = await getDMMF({
  datamodelPath: './prisma/schema.prisma',
  cwd: process.cwd(),
  prismaPath: '/custom/path/to/prisma',
  retry: 3
})
```

**getDMMF Options:**
- `datamodel?: string` - Prisma schema as a string
- `datamodelPath?: string` - Path to the Prisma schema file
- `cwd?: string` - Current working directory
- `prismaPath?: string` - Custom path to Prisma binary
- `retry?: number` - Number of retries

### 2. formatSchema

Formats a Prisma schema string according to Prisma's formatting rules.

```typescript
import { formatSchema } from '@prisma/internals'

const formattedSchema = await formatSchema({
  schema: `model   User   {
    id Int   @id
    email   String  @unique
    name String?
  }`
})

console.log(formattedSchema)
// Output: Properly formatted schema
```

### 3. getConfig

Retrieves configuration from a Prisma schema, including datasources and generators.

```typescript
import { getConfig } from '@prisma/internals'

const config = await getConfig({
  datamodel: schemaString,
  ignoreEnvVarErrors: true
})

console.log(config.datasources)
console.log(config.generators)
```

### 4. getGenerators

Gets all generators defined in the Prisma schema and prepares them for execution.

```typescript
import { getGenerators } from '@prisma/internals'

const generators = await getGenerators({
  schemaPath: './prisma/schema.prisma',
  printDownloadProgress: true,
  version: '5.0.0',
  generatorNames: ['prisma-client-js']
})
```

### 5. parseEnvValue

Parses environment variable values, supporting various formats and expansions.

```typescript
import { parseEnvValue } from '@prisma/internals'

// Simple value
const value = parseEnvValue('postgresql://localhost:5432/mydb')

// With env variable expansion
const valueWithEnv = parseEnvValue('${DATABASE_URL}')

// With complex expressions
const complex = parseEnvValue('postgresql://${DB_USER}:${DB_PASS}@localhost/mydb')
```

### 6. validate

Validates a Prisma schema for syntax and semantic errors.

```typescript
import { validate } from '@prisma/internals'

try {
  await validate({
    datamodelPath: './prisma/schema.prisma'
  })
  console.log('Schema is valid!')
} catch (error) {
  console.error('Schema validation failed:', error)
}
```

### 7. Engine Commands

Various commands for interacting with Prisma engines:

```typescript
import { 
  getVersion,
  resolveBinary,
  getSchemaPath
} from '@prisma/internals'

// Get version information
const version = await getVersion()

// Resolve binary paths
const binaryPath = await resolveBinary('query-engine')
```

## Utility Functions

### 8. Logger

Colorful logging utility used internally by Prisma.

```typescript
import { logger } from '@prisma/internals'

logger.info('Information message')
logger.warn('Warning message')
logger.error('Error message')
logger.log('Regular log message')
```

### 9. Debug

Debug utility based on the `debug` npm package, modified for Prisma's use.

```typescript
import { Debug } from '@prisma/internals'

const debug = Debug('my-generator')
debug('Processing schema...')
debug('Found %d models', modelCount)
```

### 10. Platform Detection

Utilities for detecting the current platform and architecture.

```typescript
import { getPlatform, platforms } from '@prisma/internals'

// Get current platform
const platform = await getPlatform()
console.log(platform) // e.g., 'darwin-arm64', 'windows', 'linux-musl'

// List all supported platforms
console.log(platforms)
```

### 11. Package Management

Utilities for detecting and working with package managers.

```typescript
import { getPackageManager } from '@prisma/internals'

const packageManager = await getPackageManager(process.cwd())
console.log(packageManager) // 'npm', 'yarn', 'pnpm'
```

### 12. File System Utilities

Enhanced file system operations.

```typescript
import { 
  pathExists,
  readFile,
  writeFile,
  ensureDir
} from '@prisma/internals'

// Check if path exists
const exists = await pathExists('./prisma/schema.prisma')

// Read file
const content = await readFile('./schema.prisma', 'utf-8')

// Write file
await writeFile('./output.prisma', content)

// Ensure directory exists
await ensureDir('./generated')
```

### 13. Schema File Input Handling

Utilities for handling single or multiple schema files.

```typescript
import { 
  getSchemaPath,
  getSchemaPathSync,
  getSchemaWithPath
} from '@prisma/internals'

// Get schema path
const schemaPath = await getSchemaPath(process.cwd())

// Get schema with content
const { schemaPath, schemas } = await getSchemaWithPath(process.cwd())
```

## Type Definitions and Interfaces

### DMMF Structure

The DMMF (Data Model Meta Format) is the core data structure returned by `getDMMF`:

```typescript
interface DMMF.Document {
  datamodel: DMMF.Datamodel
  schema: DMMF.Schema
  mappings: DMMF.Mappings
}

interface DMMF.Datamodel {
  models: DMMF.Model[]
  enums: DMMF.DatamodelEnum[]
  types: DMMF.Model[]
}

interface DMMF.Model {
  name: string
  dbName: string | null
  fields: DMMF.Field[]
  fieldMap?: Record<string, DMMF.Field>
  uniqueFields: string[][]
  uniqueIndexes: DMMF.UniqueIndex[]
  documentation?: string
  primaryKey: DMMF.PrimaryKey | null
  isGenerated?: boolean
}

interface DMMF.Field {
  name: string
  kind: DMMF.FieldKind
  isList: boolean
  isRequired: boolean
  isUnique: boolean
  isId: boolean
  isReadOnly: boolean
  isGenerated?: boolean
  isUpdatedAt?: boolean
  type: string
  dbName?: string | null
  hasDefaultValue: boolean
  default?: DMMF.FieldDefault | DMMF.FieldDefaultScalar | DMMF.FieldDefaultScalar[]
  relationName?: string
  relationFromFields?: string[]
  relationToFields?: string[]
  relationOnDelete?: string
  relationOnUpdate?: string
  documentation?: string
}

interface DMMF.DatamodelEnum {
  name: string
  values: DMMF.EnumValue[]
  dbName?: string | null
  documentation?: string
}
```

### Generator Types

```typescript
interface GeneratorConfig {
  name: string
  provider: string | GeneratorConfigProvider
  output: string | null
  config: Record<string, string>
  binaryTargets: BinaryTargetsEnvValue[]
  previewFeatures: string[]
}

interface GeneratorOptions {
  generator: GeneratorConfig
  otherGenerators: GeneratorConfig[]
  schemaPath: string
  dmmf: DMMF.Document
  datasources: DataSource[]
  datamodel: string
  version: string
  binaryPaths?: BinaryPaths
}
```

### Configuration Types

```typescript
interface ConfigMetaFormat {
  datasources: DataSource[]
  generators: GeneratorConfig[]
  warnings: string[]
}

interface DataSource {
  name: string
  provider: string
  url: string | { fromEnvVar: string | null; value?: string }
  activeProvider?: string
  schemas?: string[]
}
```

## Advanced Usage

### Building a Complete Prisma Generator

```typescript
import { generatorHandler } from '@prisma/generator-helper'
import { getDMMF, formatSchema, logger } from '@prisma/internals'
import * as fs from 'fs/promises'
import * as path from 'path'

generatorHandler({
  onManifest() {
    logger.info('Generator manifest called')
    return {
      prettyName: 'My Custom Generator',
      defaultOutput: './generated',
      requiresGenerators: ['prisma-client-js']
    }
  },
  async onGenerate(options) {
    logger.info('Starting generation...')
    
    // Get DMMF from the datamodel
    const dmmf = await getDMMF({
      datamodel: options.datamodel,
    })
    
    // Generate custom code based on models
    const output = options.generator.output?.value || './generated'
    await fs.mkdir(output, { recursive: true })
    
    for (const model of dmmf.datamodel.models) {
      const modelCode = generateModelCode(model)
      const fileName = path.join(output, `${model.name.toLowerCase()}.ts`)
      await fs.writeFile(fileName, modelCode)
      logger.info(`Generated ${fileName}`)
    }
    
    logger.info('Generation completed!')
  },
})

function generateModelCode(model: DMMF.Model): string {
  return `
// Generated code for ${model.name}
export interface ${model.name} {
${model.fields.map(field => `  ${field.name}: ${getTypeScriptType(field)}`).join('\n')}
}

export const ${model.name}Schema = {
  name: '${model.name}',
  fields: ${JSON.stringify(model.fields, null, 2)}
}
`
}

function getTypeScriptType(field: DMMF.Field): string {
  const baseType = field.type
  const tsType = mapPrismaTypeToTS(baseType)
  const nullable = !field.isRequired ? ' | null' : ''
  const array = field.isList ? '[]' : ''
  return `${tsType}${array}${nullable}`
}
```

### Testing with @prisma/internals

```typescript
import { getDMMF, validate } from '@prisma/internals'
import { describe, it, expect } from '@jest/globals'

describe('Schema Tests', () => {
  it('should parse schema correctly', async () => {
    const schema = `
      model User {
        id    Int    @id @default(autoincrement())
        email String @unique
        posts Post[]
      }
      
      model Post {
        id       Int    @id @default(autoincrement())
        title    String
        author   User   @relation(fields: [authorId], references: [id])
        authorId Int
      }
    `
    
    const dmmf = await getDMMF({ datamodel: schema })
    
    expect(dmmf.datamodel.models).toHaveLength(2)
    expect(dmmf.datamodel.models[0].name).toBe('User')
    expect(dmmf.datamodel.models[0].fields).toHaveLength(3)
  })
  
  it('should validate schema', async () => {
    await expect(validate({
      datamodel: `
        model User {
          id Int @id
        }
      `
    })).resolves.not.toThrow()
  })
  
  it('should throw on invalid schema', async () => {
    await expect(validate({
      datamodel: `
        model User {
          id Int
        }
      `
    })).rejects.toThrow() // Missing @id
  })
})
```

### Schema Analysis Tool

```typescript
import { getDMMF, getConfig, logger } from '@prisma/internals'
import * as fs from 'fs/promises'

async function analyzeSchema(schemaPath: string) {
  logger.info(`Analyzing schema: ${schemaPath}`)
  
  const schema = await fs.readFile(schemaPath, 'utf-8')
  const dmmf = await getDMMF({ datamodel: schema })
  const config = await getConfig({ datamodel: schema })
  
  const analysis = {
    models: dmmf.datamodel.models.length,
    enums: dmmf.datamodel.enums.length,
    fields: dmmf.datamodel.models.reduce(
      (acc, model) => acc + model.fields.length, 
      0
    ),
    relations: dmmf.datamodel.models.reduce(
      (acc, model) => acc + model.fields.filter(f => f.kind === 'object').length,
      0
    ),
    datasources: config.datasources.length,
    generators: config.generators.length,
    modelDetails: dmmf.datamodel.models.map(model => ({
      name: model.name,
      fields: model.fields.length,
      relations: model.fields.filter(f => f.kind === 'object').length,
      uniqueConstraints: model.uniqueFields.length,
      hasIdField: model.fields.some(f => f.isId),
      documentation: model.documentation
    }))
  }
  
  return analysis
}

// Usage
analyzeSchema('./prisma/schema.prisma')
  .then(analysis => {
    console.log('Schema Analysis:', JSON.stringify(analysis, null, 2))
  })
  .catch(error => {
    logger.error('Analysis failed:', error)
  })
```

### Multi-file Schema Support

```typescript
import { 
  getSchemaWithPath,
  formatSchema,
  getDMMF 
} from '@prisma/internals'

async function processMultiFileSchema(projectPath: string) {
  // Get all schema files
  const { schemas, schemaPath } = await getSchemaWithPath(projectPath)
  
  if (Array.isArray(schemas)) {
    // Multiple schema files
    logger.info(`Found ${schemas.length} schema files`)
    
    for (const schema of schemas) {
      logger.info(`Processing: ${schema.path}`)
      // Process each schema file
    }
  } else {
    // Single schema file
    const dmmf = await getDMMF({ datamodel: schemas })
    // Process single schema
  }
}
```

## Error Handling

### Common Errors and Solutions

```typescript
import { getDMMF, logger } from '@prisma/internals'

async function safeGetDMMF(schemaPath: string) {
  try {
    const dmmf = await getDMMF({ datamodelPath: schemaPath })
    return dmmf
  } catch (error) {
    if (error.message.includes('Could not find libquery-engine')) {
      logger.error('Query engine not found. Run: npx prisma generate')
    } else if (error.message.includes('Schema parsing')) {
      logger.error('Invalid schema syntax:', error.message)
    } else if (error.message.includes('Environment variable not found')) {
      logger.error('Missing environment variable:', error.message)
    } else {
      logger.error('Unknown error:', error)
    }
    throw error
  }
}
```

## Best Practices

### 1. Version Pinning

Since @prisma/internals doesn't follow semver, always pin to a specific version:

```json
{
  "dependencies": {
    "@prisma/internals": "5.0.0"
  }
}
```

### 2. Type Safety

Always use TypeScript for better type safety:

```typescript
import type { DMMF, GeneratorConfig } from '@prisma/internals'

function processModel(model: DMMF.Model): void {
  // Type-safe model processing
}
```

### 3. Error Boundaries

Wrap all calls in try-catch blocks:

```typescript
async function safeDMMF(schema: string): Promise<DMMF.Document | null> {
  try {
    return await getDMMF({ datamodel: schema })
  } catch (error) {
    console.error('Failed to parse schema:', error)
    return null
  }
}
```

### 4. Environment Handling

Use proper environment variable handling:

```typescript
import { parseEnvValue } from '@prisma/internals'

const databaseUrl = parseEnvValue(process.env.DATABASE_URL || '')
if (!databaseUrl) {
  throw new Error('DATABASE_URL is required')
}
```

### 5. Logging

Use the built-in logger for consistent output:

```typescript
import { logger } from '@prisma/internals'

logger.info('Starting generator')
logger.warn('Deprecated feature used')
logger.error('Generation failed')
```

## Migration Guide

### From @prisma/sdk to @prisma/internals

```typescript
// Old (@prisma/sdk)
import { getDMMF, formatSchema } from '@prisma/sdk'

// New (@prisma/internals)
import { getDMMF, formatSchema } from '@prisma/internals'
```

Most APIs remain the same, but be aware:
- Some exports may have been removed
- Type definitions may have changed
- Internal implementation details have changed

### Handling Breaking Changes

Create wrapper functions to isolate usage:

```typescript
// wrapper.ts
import * as internals from '@prisma/internals'

export async function getDMMF(options: any) {
  try {
    // Try new API
    return await internals.getDMMF(options)
  } catch (error) {
    // Fallback or error handling
    console.error('getDMMF failed:', error)
    throw error
  }
}
```

## Community Resources

- [Prisma Generator Hub](https://github.com/prisma/generator-hub)
- [Create Prisma Generator](https://github.com/YassinEldeeb/create-prisma-generator)
- [Prisma Community Discussions](https://github.com/prisma/prisma/discussions)
- [Prisma Discord](https://discord.gg/prisma)

## Known Limitations

1. **No Official Support**: The Prisma team does not provide support for this package
2. **Breaking Changes**: APIs can change without notice between any versions
3. **Limited Documentation**: No official documentation exists
4. **Query Engine Dependency**: Many functions require the Prisma query engine binary
5. **Platform Specific**: Some features may not work on all platforms
6. **Memory Usage**: Large schemas can consume significant memory
7. **Edge Runtime**: Not compatible with edge runtimes

## Alternatives

For production use, consider:

1. **Official Prisma Client APIs**: Use the documented Prisma Client features
2. **Generator Framework**: Use only the official `@prisma/generator-helper` package
3. **Custom Parsers**: Build your own schema parser if needed
4. **Wait for Public APIs**: Prisma may release official public APIs in the future

## Future Considerations

The Prisma team has acknowledged the community's use of this package and is considering:
- Official public APIs for common use cases
- Better documentation for generator authors
- Stable APIs for schema introspection
- Improved tooling for Prisma ecosystem development

Until then, use `@prisma/internals` with caution and be prepared for changes.

## Conclusion

While `@prisma/internals` provides powerful utilities for working with Prisma schemas programmatically, it's crucial to understand its limitations and risks. This package is best suited for:
- Development and testing environments
- Building proof-of-concept generators
- Learning and experimentation
- Tools where you can control the Prisma version

For production applications, especially those that need long-term stability, consider alternatives or ensure you have proper error handling and version pinning strategies in place.
