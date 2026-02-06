# General Storage - Code Implementation Template

This document provides code templates that can be directly used to implement the General Storage system. Each section contains ready-to-use TypeScript code.

## Core Interfaces

### File: `src/storage/interfaces/IStorageContainer.ts`

```typescript
/**
 * Main storage container interface for managing data storage operations.
 */
export interface IStorageContainer {
    /**
     * Initialize the storage container and its backend.
     * Must be called before any other operations.
     */
    initialize(): Promise<void>;

    /**
     * Store a value in storage.
     * @param key - Unique identifier for the stored value
     * @param value - The value to store
     * @param options - Optional storage configuration
     */
    set<T>(key: string, value: T, options?: StorageOptions): Promise<void>;

    /**
     * Retrieve a value from storage.
     * @param key - The key to retrieve
     * @returns The stored value or null if not found
     */
    get<T>(key: string): Promise<T | null>;

    /**
     * Check if a key exists in storage.
     * @param key - The key to check
     * @returns True if the key exists, false otherwise
     */
    has(key: string): Promise<boolean>;

    /**
     * Remove a value from storage.
     * @param key - The key to remove
     */
    remove(key: string): Promise<void>;

    /**
     * Remove all values from storage.
     */
    clear(): Promise<void>;

    /**
     * Get all keys currently stored.
     * @returns Array of all keys
     */
    keys(): Promise<string[]>;

    /**
     * Get storage statistics and metrics.
     * @returns Storage statistics object
     */
    getStats(): Promise<StorageStats>;

    /**
     * Dispose of the storage container and clean up resources.
     */
    dispose(): Promise<void>;
}
```

### File: `src/storage/interfaces/IStorageBackend.ts`

```typescript
import { SerializedData } from '../types/SerializedData';

/**
 * Backend interface for different storage implementations.
 */
export interface IStorageBackend {
    /**
     * Initialize the backend.
     */
    initialize(): Promise<void>;

    /**
     * Write serialized data to the backend.
     * @param key - The storage key
     * @param data - The serialized data to write
     */
    write(key: string, data: SerializedData): Promise<void>;

    /**
     * Read serialized data from the backend.
     * @param key - The storage key
     * @returns The serialized data or null if not found
     */
    read(key: string): Promise<SerializedData | null>;

    /**
     * Delete data from the backend.
     * @param key - The storage key to delete
     */
    delete(key: string): Promise<void>;

    /**
     * List all keys in the backend.
     * @returns Array of all keys
     */
    list(): Promise<string[]>;

    /**
     * Get the size of stored data in bytes.
     * @returns Total size in bytes
     */
    getSize(): Promise<number>;

    /**
     * Dispose of the backend and clean up resources.
     */
    dispose(): Promise<void>;
}
```

### File: `src/storage/interfaces/ISerializer.ts`

```typescript
import { SerializedData, SerializationMetadata } from '../types/SerializedData';

/**
 * Serializer interface for converting data to/from storage format.
 */
export interface ISerializer {
    /**
     * Serialize data to storage format.
     * @param data - The data to serialize
     * @param metadata - Optional metadata for serialization
     * @returns Serialized data with metadata
     */
    serialize<T>(data: T, metadata?: SerializationMetadata): SerializedData;

    /**
     * Deserialize data from storage format.
     * @param data - The serialized data
     * @param metadata - Optional metadata for deserialization
     * @returns The original data
     */
    deserialize<T>(data: SerializedData, metadata?: SerializationMetadata): T;

    /**
     * Check if the serializer can handle the given data.
     * @param data - The data to check
     * @returns True if serializable, false otherwise
     */
    canSerialize(data: any): boolean;
}
```

## Type Definitions

### File: `src/storage/types/StorageOptions.ts`

```typescript
/**
 * Options for storage operations.
 */
export interface StorageOptions {
    /**
     * Time to live in milliseconds. After this time, the item will be considered expired.
     */
    ttl?: number;

    /**
     * Enable compression for the stored data.
     */
    compress?: boolean;

    /**
     * Enable encryption for the stored data.
     */
    encrypt?: boolean;

    /**
     * Custom metadata to store with the data.
     */
    metadata?: Record<string, any>;

    /**
     * Priority level for caching decisions.
     */
    priority?: 'low' | 'medium' | 'high';

    /**
     * Tags for categorizing stored items.
     */
    tags?: string[];
}
```

### File: `src/storage/types/SerializedData.ts`

```typescript
/**
 * Represents serialized data with metadata.
 */
export interface SerializedData {
    /**
     * The actual serialized data (string or binary).
     */
    data: string | Buffer;

    /**
     * Type identifier for deserialization.
     */
    type: string;

    /**
     * Serialization format version.
     */
    version: string;

    /**
     * Timestamp when the data was serialized.
     */
    timestamp: number;

    /**
     * Optional checksum for data validation.
     */
    checksum?: string;

    /**
     * Optional compression algorithm used.
     */
    compression?: 'gzip' | 'deflate' | 'brotli' | 'none';

    /**
     * Optional encryption algorithm used.
     */
    encryption?: 'aes-256-gcm' | 'none';
}

/**
 * Metadata for serialization operations.
 */
export interface SerializationMetadata {
    /**
     * Expected type name for validation.
     */
    expectedType?: string;

    /**
     * Whether to validate checksums.
     */
    validateChecksum?: boolean;

    /**
     * Custom serialization options.
     */
    options?: Record<string, any>;
}
```

### File: `src/storage/types/StorageStats.ts`

```typescript
/**
 * Storage statistics and metrics.
 */
export interface StorageStats {
    /**
     * Total number of items stored.
     */
    itemCount: number;

    /**
     * Total size of stored data in bytes.
     */
    totalSize: number;

    /**
     * Type of backend in use.
     */
    backendType: string;

    /**
     * Timestamp of last storage access.
     */
    lastAccess: number;

    /**
     * Cache hit rate (0-1) if caching is enabled.
     */
    hitRate?: number;

    /**
     * Number of expired items (if TTL is enabled).
     */
    expiredCount?: number;

    /**
     * Available storage space in bytes.
     */
    availableSpace?: number;
}
```

### File: `src/storage/types/BackendOptions.ts`

```typescript
/**
 * Options for MemoryBackend.
 */
export interface MemoryBackendOptions {
    /**
     * Maximum size in bytes for the memory storage.
     */
    maxSize?: number;

    /**
     * Eviction policy when max size is reached.
     */
    evictionPolicy?: 'lru' | 'lfu' | 'fifo';
}

/**
 * Options for FileSystemBackend.
 */
export interface FileSystemBackendOptions {
    /**
     * Base directory path for file storage.
     */
    basePath: string;

    /**
     * Size of the in-memory cache.
     */
    cacheSize?: number;

    /**
     * File extension for stored files.
     */
    fileExtension?: string;

    /**
     * Enable atomic writes.
     */
    atomicWrites?: boolean;
}

/**
 * Options for IndexedDBBackend.
 */
export interface IndexedDBBackendOptions {
    /**
     * Name of the IndexedDB database.
     */
    dbName: string;

    /**
     * Database version number.
     */
    version?: number;

    /**
     * Name of the object store.
     */
    storeName?: string;
}

/**
 * Options for CachedBackend.
 */
export interface CachedBackendOptions {
    /**
     * Primary storage backend.
     */
    primary: IStorageBackend;

    /**
     * Cache storage backend.
     */
    cache: IStorageBackend;

    /**
     * Cache eviction policy.
     */
    evictionPolicy?: 'lru' | 'lfu' | 'fifo';

    /**
     * Maximum cache size in bytes.
     */
    maxCacheSize?: number;

    /**
     * Write-through cache enabled.
     */
    writeThrough?: boolean;
}
```

## Error Classes

### File: `src/storage/errors/StorageError.ts`

```typescript
/**
 * Base class for storage-related errors.
 */
export class StorageError extends Error {
    constructor(message: string, public readonly code: string) {
        super(message);
        this.name = 'StorageError';
        Object.setPrototypeOf(this, StorageError.prototype);
    }
}

/**
 * Error thrown when a key is not found in storage.
 */
export class KeyNotFoundError extends StorageError {
    constructor(key: string) {
        super(`Key not found: ${key}`, 'KEY_NOT_FOUND');
        this.name = 'KeyNotFoundError';
        Object.setPrototypeOf(this, KeyNotFoundError.prototype);
    }
}

/**
 * Error thrown when serialization fails.
 */
export class SerializationError extends StorageError {
    constructor(message: string, public readonly originalError?: Error) {
        super(`Serialization failed: ${message}`, 'SERIALIZATION_ERROR');
        this.name = 'SerializationError';
        Object.setPrototypeOf(this, SerializationError.prototype);
    }
}

/**
 * Error thrown when deserialization fails.
 */
export class DeserializationError extends StorageError {
    constructor(message: string, public readonly originalError?: Error) {
        super(`Deserialization failed: ${message}`, 'DESERIALIZATION_ERROR');
        this.name = 'DeserializationError';
        Object.setPrototypeOf(this, DeserializationError.prototype);
    }
}

/**
 * Error thrown when backend operations fail.
 */
export class BackendError extends StorageError {
    constructor(message: string, public readonly originalError?: Error) {
        super(`Backend error: ${message}`, 'BACKEND_ERROR');
        this.name = 'BackendError';
        Object.setPrototypeOf(this, BackendError.prototype);
    }
}

/**
 * Error thrown when storage quota is exceeded.
 */
export class StorageQuotaExceededError extends StorageError {
    constructor(public readonly requestedSize?: number, public readonly availableSize?: number) {
        super('Storage quota exceeded', 'QUOTA_EXCEEDED');
        this.name = 'StorageQuotaExceededError';
        Object.setPrototypeOf(this, StorageQuotaExceededError.prototype);
    }
}

/**
 * Error thrown when data validation fails.
 */
export class ValidationError extends StorageError {
    constructor(message: string) {
        super(`Validation failed: ${message}`, 'VALIDATION_ERROR');
        this.name = 'ValidationError';
        Object.setPrototypeOf(this, ValidationError.prototype);
    }
}

/**
 * Error thrown when initialization fails.
 */
export class InitializationError extends StorageError {
    constructor(message: string, public readonly originalError?: Error) {
        super(`Initialization failed: ${message}`, 'INITIALIZATION_ERROR');
        this.name = 'InitializationError';
        Object.setPrototypeOf(this, InitializationError.prototype);
    }
}
```

## Main Implementation Templates

### File: `src/storage/StorageContainer.ts`

```typescript
import { IStorageContainer } from './interfaces/IStorageContainer';
import { IStorageBackend } from './interfaces/IStorageBackend';
import { ISerializer } from './interfaces/ISerializer';
import { StorageOptions } from './types/StorageOptions';
import { StorageStats } from './types/StorageStats';
import { KeyNotFoundError } from './errors/StorageError';

/**
 * Configuration for StorageContainer.
 */
export interface StorageContainerConfig {
    backend: IStorageBackend;
    serializer: ISerializer;
    defaultOptions?: StorageOptions;
    debug?: boolean;
    logger?: Console;
}

/**
 * Main storage container implementation.
 */
export class StorageContainer implements IStorageContainer {
    private backend: IStorageBackend;
    private serializer: ISerializer;
    private defaultOptions: StorageOptions;
    private initialized: boolean = false;
    private debug: boolean;
    private logger: Console;

    constructor(config: StorageContainerConfig) {
        this.backend = config.backend;
        this.serializer = config.serializer;
        this.defaultOptions = config.defaultOptions || {};
        this.debug = config.debug || false;
        this.logger = config.logger || console;
    }

    async initialize(): Promise<void> {
        if (this.initialized) {
            return;
        }

        try {
            await this.backend.initialize();
            this.initialized = true;
            this.log('Storage container initialized');
        } catch (error) {
            throw new InitializationError(
                'Failed to initialize storage container',
                error as Error
            );
        }
    }

    async set<T>(key: string, value: T, options?: StorageOptions): Promise<void> {
        this.ensureInitialized();
        this.validateKey(key);

        const mergedOptions = { ...this.defaultOptions, ...options };
        
        try {
            // Serialize the data
            const serialized = this.serializer.serialize(value);
            
            // Write to backend
            await this.backend.write(key, serialized);
            
            this.log(`Set key: ${key}`);
        } catch (error) {
            throw new BackendError(`Failed to set key: ${key}`, error as Error);
        }
    }

    async get<T>(key: string): Promise<T | null> {
        this.ensureInitialized();
        this.validateKey(key);

        try {
            // Read from backend
            const serialized = await this.backend.read(key);
            
            if (serialized === null) {
                return null;
            }

            // Deserialize the data
            const value = this.serializer.deserialize<T>(serialized);
            
            this.log(`Get key: ${key}`);
            return value;
        } catch (error) {
            throw new BackendError(`Failed to get key: ${key}`, error as Error);
        }
    }

    async has(key: string): Promise<boolean> {
        this.ensureInitialized();
        this.validateKey(key);

        try {
            const data = await this.backend.read(key);
            return data !== null;
        } catch (error) {
            throw new BackendError(`Failed to check key: ${key}`, error as Error);
        }
    }

    async remove(key: string): Promise<void> {
        this.ensureInitialized();
        this.validateKey(key);

        try {
            await this.backend.delete(key);
            this.log(`Removed key: ${key}`);
        } catch (error) {
            throw new BackendError(`Failed to remove key: ${key}`, error as Error);
        }
    }

    async clear(): Promise<void> {
        this.ensureInitialized();

        try {
            const keys = await this.backend.list();
            await Promise.all(keys.map(key => this.backend.delete(key)));
            this.log('Cleared all storage');
        } catch (error) {
            throw new BackendError('Failed to clear storage', error as Error);
        }
    }

    async keys(): Promise<string[]> {
        this.ensureInitialized();

        try {
            return await this.backend.list();
        } catch (error) {
            throw new BackendError('Failed to list keys', error as Error);
        }
    }

    async getStats(): Promise<StorageStats> {
        this.ensureInitialized();

        try {
            const keys = await this.backend.list();
            const totalSize = await this.backend.getSize();

            return {
                itemCount: keys.length,
                totalSize,
                backendType: this.backend.constructor.name,
                lastAccess: Date.now()
            };
        } catch (error) {
            throw new BackendError('Failed to get stats', error as Error);
        }
    }

    async dispose(): Promise<void> {
        if (!this.initialized) {
            return;
        }

        try {
            await this.backend.dispose();
            this.initialized = false;
            this.log('Storage container disposed');
        } catch (error) {
            throw new BackendError('Failed to dispose storage', error as Error);
        }
    }

    private ensureInitialized(): void {
        if (!this.initialized) {
            throw new Error('Storage container not initialized. Call initialize() first.');
        }
    }

    private validateKey(key: string): void {
        if (!key || typeof key !== 'string') {
            throw new ValidationError('Key must be a non-empty string');
        }
        if (key.includes('\0')) {
            throw new ValidationError('Key cannot contain null bytes');
        }
    }

    private log(message: string): void {
        if (this.debug) {
            this.logger.log(`[StorageContainer] ${message}`);
        }
    }
}
```

### File: `src/storage/backends/MemoryBackend.ts`

```typescript
import { IStorageBackend } from '../interfaces/IStorageBackend';
import { SerializedData } from '../types/SerializedData';
import { MemoryBackendOptions } from '../types/BackendOptions';
import { StorageQuotaExceededError } from '../errors/StorageError';

/**
 * In-memory storage backend.
 */
export class MemoryBackend implements IStorageBackend {
    private storage: Map<string, SerializedData> = new Map();
    private maxSize: number;
    private currentSize: number = 0;

    constructor(options: MemoryBackendOptions = {}) {
        this.maxSize = options.maxSize || Infinity;
    }

    async initialize(): Promise<void> {
        // No initialization needed for memory backend
    }

    async write(key: string, data: SerializedData): Promise<void> {
        const dataSize = this.getDataSize(data);

        // Check if we need to make space
        if (this.storage.has(key)) {
            const existingSize = this.getDataSize(this.storage.get(key)!);
            this.currentSize -= existingSize;
        }

        // Check quota
        if (this.currentSize + dataSize > this.maxSize) {
            throw new StorageQuotaExceededError(dataSize, this.maxSize - this.currentSize);
        }

        this.storage.set(key, data);
        this.currentSize += dataSize;
    }

    async read(key: string): Promise<SerializedData | null> {
        return this.storage.get(key) || null;
    }

    async delete(key: string): Promise<void> {
        const data = this.storage.get(key);
        if (data) {
            this.currentSize -= this.getDataSize(data);
            this.storage.delete(key);
        }
    }

    async list(): Promise<string[]> {
        return Array.from(this.storage.keys());
    }

    async getSize(): Promise<number> {
        return this.currentSize;
    }

    async dispose(): Promise<void> {
        this.storage.clear();
        this.currentSize = 0;
    }

    private getDataSize(data: SerializedData): number {
        if (typeof data.data === 'string') {
            return data.data.length * 2; // Approximate UTF-16 size
        } else {
            return data.data.length;
        }
    }
}
```

### File: `src/storage/serializers/JSONSerializer.ts`

```typescript
import { ISerializer } from '../interfaces/ISerializer';
import { SerializedData, SerializationMetadata } from '../types/SerializedData';
import { SerializationError, DeserializationError } from '../errors/StorageError';

/**
 * JSON-based serializer.
 */
export class JSONSerializer implements ISerializer {
    private readonly version = '1.0.0';

    serialize<T>(data: T, metadata?: SerializationMetadata): SerializedData {
        try {
            const jsonString = JSON.stringify(data);
            
            return {
                data: jsonString,
                type: typeof data === 'object' && data !== null 
                    ? (data as any).constructor.name 
                    : typeof data,
                version: this.version,
                timestamp: Date.now(),
                compression: 'none',
                encryption: 'none'
            };
        } catch (error) {
            throw new SerializationError(
                'Failed to serialize data to JSON',
                error as Error
            );
        }
    }

    deserialize<T>(data: SerializedData, metadata?: SerializationMetadata): T {
        try {
            if (typeof data.data !== 'string') {
                throw new Error('Expected string data for JSON deserialization');
            }

            return JSON.parse(data.data) as T;
        } catch (error) {
            throw new DeserializationError(
                'Failed to deserialize JSON data',
                error as Error
            );
        }
    }

    canSerialize(data: any): boolean {
        try {
            JSON.stringify(data);
            return true;
        } catch {
            return false;
        }
    }
}
```

### File: `src/storage/index.ts`

```typescript
// Main exports
export { StorageContainer } from './StorageContainer';

// Interfaces
export { IStorageContainer } from './interfaces/IStorageContainer';
export { IStorageBackend } from './interfaces/IStorageBackend';
export { ISerializer } from './interfaces/ISerializer';

// Backends
export { MemoryBackend } from './backends/MemoryBackend';
// export { FileSystemBackend } from './backends/FileSystemBackend';
// export { IndexedDBBackend } from './backends/IndexedDBBackend';
// export { CachedBackend } from './backends/CachedBackend';

// Serializers
export { JSONSerializer } from './serializers/JSONSerializer';
// export { BinarySerializer } from './serializers/BinarySerializer';
// export { MessagePackSerializer } from './serializers/MessagePackSerializer';

// Types
export { StorageOptions } from './types/StorageOptions';
export { SerializedData, SerializationMetadata } from './types/SerializedData';
export { StorageStats } from './types/StorageStats';
export {
    MemoryBackendOptions,
    FileSystemBackendOptions,
    IndexedDBBackendOptions,
    CachedBackendOptions
} from './types/BackendOptions';

// Errors
export {
    StorageError,
    KeyNotFoundError,
    SerializationError,
    DeserializationError,
    BackendError,
    StorageQuotaExceededError,
    ValidationError,
    InitializationError
} from './errors/StorageError';
```

## Test Templates

### File: `tests/unit/StorageContainer.test.ts`

```typescript
import { describe, it, expect, beforeEach, afterEach } from '@jest/globals';
import { StorageContainer } from '../../src/storage/StorageContainer';
import { MemoryBackend } from '../../src/storage/backends/MemoryBackend';
import { JSONSerializer } from '../../src/storage/serializers/JSONSerializer';

describe('StorageContainer', () => {
    let storage: StorageContainer;

    beforeEach(async () => {
        storage = new StorageContainer({
            backend: new MemoryBackend(),
            serializer: new JSONSerializer()
        });
        await storage.initialize();
    });

    afterEach(async () => {
        await storage.dispose();
    });

    describe('basic operations', () => {
        it('should store and retrieve a value', async () => {
            await storage.set('test', 'value');
            const result = await storage.get('test');
            expect(result).toBe('value');
        });

        it('should return null for non-existent keys', async () => {
            const result = await storage.get('nonexistent');
            expect(result).toBeNull();
        });

        it('should check if key exists', async () => {
            await storage.set('test', 'value');
            expect(await storage.has('test')).toBe(true);
            expect(await storage.has('nonexistent')).toBe(false);
        });

        it('should remove a value', async () => {
            await storage.set('test', 'value');
            await storage.remove('test');
            expect(await storage.has('test')).toBe(false);
        });

        it('should clear all values', async () => {
            await storage.set('key1', 'value1');
            await storage.set('key2', 'value2');
            await storage.clear();
            expect(await storage.keys()).toHaveLength(0);
        });

        it('should list all keys', async () => {
            await storage.set('key1', 'value1');
            await storage.set('key2', 'value2');
            const keys = await storage.keys();
            expect(keys).toContain('key1');
            expect(keys).toContain('key2');
        });
    });

    describe('type safety', () => {
        it('should handle objects', async () => {
            const obj = { foo: 'bar', num: 42 };
            await storage.set('obj', obj);
            const result = await storage.get<typeof obj>('obj');
            expect(result).toEqual(obj);
        });

        it('should handle arrays', async () => {
            const arr = [1, 2, 3];
            await storage.set('arr', arr);
            const result = await storage.get<number[]>('arr');
            expect(result).toEqual(arr);
        });

        it('should handle nested structures', async () => {
            const nested = {
                users: [
                    { id: 1, name: 'Alice' },
                    { id: 2, name: 'Bob' }
                ],
                metadata: { version: '1.0' }
            };
            await storage.set('nested', nested);
            const result = await storage.get<typeof nested>('nested');
            expect(result).toEqual(nested);
        });
    });

    describe('statistics', () => {
        it('should return storage stats', async () => {
            await storage.set('key1', 'value1');
            await storage.set('key2', 'value2');
            const stats = await storage.getStats();
            expect(stats.itemCount).toBe(2);
            expect(stats.totalSize).toBeGreaterThan(0);
        });
    });
});
```

## Package.json Template

### File: `package.json`

```json
{
  "name": "@samsara/storage",
  "version": "1.0.0",
  "description": "General storage system for Samsara Engine",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/**/*.ts",
    "format": "prettier --write \"src/**/*.ts\"",
    "clean": "rm -rf dist"
  },
  "keywords": [
    "storage",
    "database",
    "persistence",
    "samsara",
    "engine"
  ],
  "author": "Samsara Engine Team",
  "license": "MIT",
  "devDependencies": {
    "@types/jest": "^29.0.0",
    "@types/node": "^20.0.0",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.0.0",
    "jest": "^29.0.0",
    "prettier": "^3.0.0",
    "ts-jest": "^29.0.0",
    "typescript": "^5.0.0"
  },
  "dependencies": {}
}
```

## TypeScript Configuration

### File: `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "types": ["node", "jest"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

## Implementation Checklist

- [ ] Set up project structure
- [ ] Install dependencies (`npm install`)
- [ ] Implement core interfaces
- [ ] Implement type definitions
- [ ] Implement error classes
- [ ] Implement StorageContainer
- [ ] Implement MemoryBackend
- [ ] Implement JSONSerializer
- [ ] Write unit tests
- [ ] Verify all tests pass
- [ ] Build the project (`npm run build`)
- [ ] Generate documentation
- [ ] Review and optimize
- [ ] Prepare for Phase 2 (FileSystemBackend, etc.)

## Next Steps

After implementing the core system, proceed with:

1. **Phase 2**: FileSystemBackend and persistence
2. **Phase 3**: Advanced features (compression, encryption, caching)
3. **Phase 4**: Browser support (IndexedDBBackend)
4. **Phase 5**: Additional serializers (Binary, MessagePack)
5. **Phase 6**: Performance optimizations and benchmarks
