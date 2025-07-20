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

## Key Exports and APIs

### 1. getDMMF (Get Data Model Meta Format)

The most commonly used function from this package. It parses a Prisma schema and returns the DMMF (Data Model Meta Format), which is an AST (Abstract Syntax Tree) representation of your Prisma schema.

```typescript
import { getDMMF } from '@prisma/internals'

// Basic usage
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
    }
  `
})
```

**getDMMF Options:**
- `datamodel?: string` - Prisma schema as a string
- `datamodelPath?: string` - Path to the Prisma schema file
- `cwd?: string` - Current working directory
- `prismaPath?: string` - Custom path to Prisma binary
- `retry?: number` - Number of retries

**Common Issues:**
- May throw errors about missing query engine binary
- Requires proper Prisma setup and dependencies

### 2. formatSchema

Formats a Prisma schema string according to Prisma's formatting rules.

```typescript
import { formatSchema } from '@prisma/internals'

const formattedSchema = await formatSchema({
  schema: `model   User   {
    id Int   @id
    email   String  @unique
  }`
})
```

### 3. Engine Commands

Various commands for interacting with Prisma engines:

```typescript
import { 
  getConfig,
  getVersion,
  validate
} from '@prisma/internals'
```

### 4. Generator Helper Types

Types and interfaces used when building Prisma generators:

```typescript
import type { 
  DMMF,
  GeneratorConfig,
  DataSource 
} from '@prisma/internals'
```

## Common Use Cases

### 1. Building Prisma Generators

```typescript
import { generatorHandler } from '@prisma/generator-helper'
import { getDMMF } from '@prisma/internals'

generatorHandler({
  onManifest: () => ({
    prettyName: 'My Custom Generator',
    defaultOutput: './generated',
  }),
  onGenerate: async (options) => {
    // Get DMMF from the datamodel
    const dmmf = await getDMMF({
      datamodel: options.datamodel,
    })
    
    // Access models
    const models = dmmf.datamodel.models
    
    // Generate custom code based on models
    models.forEach(model => {
      console.log(`Model: ${model.name}`)
      model.fields.forEach(field => {
        console.log(`  Field: ${field.name} (${field.type})`)
      })
    })
  },
})
```

### 2. Testing Prisma Generators

```typescript
import { getDMMF } from '@prisma/internals'
import fs from 'fs'

describe('Generator Tests', () => {
  it('should parse schema correctly', async () => {
    const schema = fs.readFileSync('./test/fixtures/schema.prisma', 'utf-8')
    const dmmf = await getDMMF({ datamodel: schema })
    
    expect(dmmf.datamodel.models).toHaveLength(2)
    expect(dmmf.datamodel.models[0].name).toBe('User')
  })
})
```

### 3. Analyzing Prisma Schemas

```typescript
import { getDMMF } from '@prisma/internals'

async function analyzeSchema(schemaPath: string) {
  const dmmf = await getDMMF({ datamodelPath: schemaPath })
  
  const stats = {
    models: dmmf.datamodel.models.length,
    enums: dmmf.datamodel.enums.length,
    totalFields: dmmf.datamodel.models.reduce(
      (acc, model) => acc + model.fields.length, 
      0
    ),
  }
  
  return stats
}
```

## DMMF Structure

The DMMF (Data Model Meta Format) returned by `getDMMF` has the following structure:

```typescript
interface DMMF.Document {
  datamodel: {
    models: Model[]
    enums: Enum[]
    types: Model[]
  }
  schema: {
    // Schema information
  }
  mappings: {
    // Mapping information
  }
}

interface Model {
  name: string
  dbName: string | null
  fields: Field[]
  primaryKey: PrimaryKey | null
  uniqueFields: string[][]
  uniqueIndexes: UniqueIndex[]
  isGenerated: boolean
  documentation?: string
  @@map?: string
}

interface Field {
  name: string
  kind: 'scalar' | 'object' | 'enum' | 'unsupported'
  type: string
  isRequired: boolean
  isList: boolean
  isUnique: boolean
  isId: boolean
  isReadOnly: boolean
  hasDefaultValue: boolean
  default?: any
  documentation?: string
  relationName?: string
  relationFromFields?: string[]
  relationToFields?: string[]
}
```

## Utilities and Helpers

### Platform Detection

```typescript
import { getPlatform } from '@prisma/internals'

const platform = await getPlatform()
console.log(platform) // e.g., 'darwin-arm64'
```

### Debug Utilities

```typescript
import { Debug } from '@prisma/internals'

const debug = Debug('my-generator')
debug('Processing schema...')
```

## Migration from @prisma/sdk

If you're migrating from the old `@prisma/sdk` package:

```typescript
// Old
import { getDMMF } from '@prisma/sdk'

// New
import { getDMMF } from '@prisma/internals'
```

Most APIs remain the same, but be aware that:
1. Some exports may have been removed or changed
2. The package location in node_modules has changed
3. Type definitions may have been updated

## Best Practices

1. **Pin Version**: Since this package doesn't follow semver, pin to a specific version:
   ```json
   {
     "dependencies": {
       "@prisma/internals": "4.16.2"
     }
   }
   ```

2. **Error Handling**: Always wrap calls in try-catch blocks:
   ```typescript
   try {
     const dmmf = await getDMMF({ datamodelPath: './schema.prisma' })
   } catch (error) {
     console.error('Failed to parse schema:', error)
   }
   ```

3. **Type Safety**: Use TypeScript for better type safety:
   ```typescript
   import type { DMMF } from '@prisma/internals'
   
   function processModel(model: DMMF.Model) {
     // Type-safe model processing
   }
   ```

4. **Testing**: Mock or stub the functions when testing:
   ```typescript
   jest.mock('@prisma/internals', () => ({
     getDMMF: jest.fn().mockResolvedValue(mockDMMF)
   }))
   ```

## Known Issues and Limitations

1. **Query Engine Binary**: getDMMF requires the Prisma query engine binary, which may cause issues in certain environments
2. **Memory Usage**: Large schemas can consume significant memory
3. **Breaking Changes**: APIs can change without notice between versions
4. **Documentation**: No official documentation exists
5. **Support**: No official support from Prisma team

## Alternatives

For production use, consider:
1. Using the official Prisma Client APIs
2. Building custom parsers for Prisma schemas
3. Using the generator framework with official APIs only
4. Waiting for official public APIs from Prisma

## Community Resources

- [Prisma Generator Hub](https://github.com/prisma/generator-hub)
- [Create Prisma Generator](https://github.com/YassinEldeeb/create-prisma-generator)
- [Prisma Community Discussions](https://github.com/prisma/prisma/discussions)

## Conclusion

While `@prisma/internals` provides powerful utilities for working with Prisma schemas programmatically, it's important to understand its limitations and risks. Use it cautiously, especially in production environments, and always be prepared for breaking changes when updating Prisma versions.
