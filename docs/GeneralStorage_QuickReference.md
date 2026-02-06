# General Storage - Quick Reference Guide

## Quick Start

### Installation
```bash
npm install @samsara/storage
# or
yarn add @samsara/storage
```

### Basic Setup
```typescript
import { StorageContainer, MemoryBackend, JSONSerializer } from '@samsara/storage';

const storage = new StorageContainer({
    backend: new MemoryBackend(),
    serializer: new JSONSerializer()
});

await storage.initialize();
```

## Common Operations

### Store Data
```typescript
// Simple value
await storage.set('key', 'value');

// Complex object
await storage.set('user', { id: 1, name: 'John' });

// With options
await storage.set('session', sessionData, {
    ttl: 3600000,  // 1 hour
    compress: true
});
```

### Retrieve Data
```typescript
// Get value
const value = await storage.get('key');

// Get with type
const user = await storage.get<User>('user');

// Check existence first
if (await storage.has('key')) {
    const value = await storage.get('key');
}
```

### Delete Data
```typescript
// Remove single item
await storage.remove('key');

// Clear all
await storage.clear();
```

### List Keys
```typescript
const keys = await storage.keys();
console.log('Stored keys:', keys);
```

## Backend Options

### Memory Backend
```typescript
const memoryBackend = new MemoryBackend({
    maxSize: 100 * 1024 * 1024  // 100MB
});
```

### FileSystem Backend
```typescript
const fsBackend = new FileSystemBackend({
    basePath: './storage',
    cacheSize: 1000
});
```

### IndexedDB Backend (Browser)
```typescript
const idbBackend = new IndexedDBBackend({
    dbName: 'samsara-storage',
    version: 1
});
```

### Cached Backend
```typescript
const cachedBackend = new CachedBackend({
    primary: new FileSystemBackend({ basePath: './data' }),
    cache: new MemoryBackend({ maxSize: 10 * 1024 * 1024 }),
    evictionPolicy: 'lru'
});
```

## Serializer Options

### JSON Serializer
```typescript
const jsonSerializer = new JSONSerializer({
    pretty: false,
    dateFormat: 'iso'
});
```

### Binary Serializer
```typescript
const binarySerializer = new BinarySerializer({
    compression: true
});
```

### MessagePack Serializer
```typescript
const msgpackSerializer = new MessagePackSerializer({
    extensionCodec: customCodec
});
```

## Advanced Patterns

### Namespaced Storage
```typescript
class NamespacedStorage {
    constructor(
        private storage: IStorageContainer,
        private namespace: string
    ) {}
    
    private getKey(key: string): string {
        return `${this.namespace}:${key}`;
    }
    
    async set(key: string, value: any): Promise<void> {
        await this.storage.set(this.getKey(key), value);
    }
    
    async get<T>(key: string): Promise<T | null> {
        return this.storage.get<T>(this.getKey(key));
    }
}

// Usage
const userStorage = new NamespacedStorage(storage, 'users');
await userStorage.set('john', userData);
```

### Batch Operations
```typescript
// Batch set
async function batchSet(
    storage: IStorageContainer,
    items: Record<string, any>
): Promise<void> {
    const promises = Object.entries(items).map(
        ([key, value]) => storage.set(key, value)
    );
    await Promise.all(promises);
}

// Batch get
async function batchGet<T>(
    storage: IStorageContainer,
    keys: string[]
): Promise<Map<string, T>> {
    const results = await Promise.all(
        keys.map(key => storage.get<T>(key))
    );
    return new Map(
        keys.map((key, i) => [key, results[i]]).filter(([, v]) => v !== null)
    );
}
```

### Storage Events
```typescript
class ObservableStorage {
    private listeners = new Map<string, Set<Function>>();
    
    constructor(private storage: IStorageContainer) {}
    
    async set(key: string, value: any): Promise<void> {
        await this.storage.set(key, value);
        this.emit('set', key, value);
    }
    
    on(event: string, callback: Function): void {
        if (!this.listeners.has(event)) {
            this.listeners.set(event, new Set());
        }
        this.listeners.get(event)!.add(callback);
    }
    
    private emit(event: string, ...args: any[]): void {
        this.listeners.get(event)?.forEach(cb => cb(...args));
    }
}
```

### TTL Management
```typescript
class TTLManager {
    private expirations = new Map<string, NodeJS.Timeout>();
    
    constructor(private storage: IStorageContainer) {}
    
    async set(
        key: string,
        value: any,
        ttl: number
    ): Promise<void> {
        await this.storage.set(key, value);
        
        // Clear existing timeout
        if (this.expirations.has(key)) {
            clearTimeout(this.expirations.get(key)!);
        }
        
        // Set new timeout
        const timeout = setTimeout(async () => {
            await this.storage.remove(key);
            this.expirations.delete(key);
        }, ttl);
        
        this.expirations.set(key, timeout);
    }
}
```

## Configuration Examples

### Development Configuration
```typescript
const devConfig: StorageConfig = {
    backend: 'memory',
    backendOptions: {
        memory: { maxSize: 50 * 1024 * 1024 }
    },
    serializationFormat: 'json',
    compression: false,
    encryption: false,
    caching: {
        enabled: false,
        maxCacheSize: 0,
        evictionPolicy: 'lru'
    }
};
```

### Production Configuration
```typescript
const prodConfig: StorageConfig = {
    backend: 'filesystem',
    backendOptions: {
        filesystem: {
            basePath: '/var/lib/samsara/storage',
            cacheSize: 10000
        }
    },
    serializationFormat: 'messagepack',
    compression: true,
    encryption: true,
    defaultTTL: 86400000, // 24 hours
    maxSize: 10 * 1024 * 1024 * 1024, // 10GB
    caching: {
        enabled: true,
        maxCacheSize: 100 * 1024 * 1024, // 100MB
        evictionPolicy: 'lru'
    }
};
```

### Browser Configuration
```typescript
const browserConfig: StorageConfig = {
    backend: 'indexeddb',
    backendOptions: {
        indexeddb: {
            dbName: 'samsara-engine',
            version: 1
        }
    },
    serializationFormat: 'json',
    compression: true,
    encryption: false,
    caching: {
        enabled: true,
        maxCacheSize: 10 * 1024 * 1024, // 10MB
        evictionPolicy: 'lru'
    }
};
```

## Error Handling

### Try-Catch Pattern
```typescript
try {
    await storage.set('key', value);
} catch (error) {
    if (error instanceof StorageQuotaExceededError) {
        console.error('Storage full');
        // Handle quota exceeded
    } else if (error instanceof SerializationError) {
        console.error('Cannot serialize data');
        // Handle serialization error
    } else {
        console.error('Storage error:', error);
        // Handle general error
    }
}
```

### Graceful Degradation
```typescript
async function safeGet<T>(
    storage: IStorageContainer,
    key: string,
    defaultValue: T
): Promise<T> {
    try {
        const value = await storage.get<T>(key);
        return value ?? defaultValue;
    } catch (error) {
        console.warn('Failed to get from storage:', error);
        return defaultValue;
    }
}
```

## Performance Tips

1. **Use Batch Operations**: Group multiple operations together
2. **Enable Compression**: For large objects (> 1KB)
3. **Use Appropriate Backend**: Memory for temp data, FS for persistence
4. **Implement Caching**: Use CachedBackend for frequently accessed data
5. **Set Reasonable TTLs**: Automatically expire stale data
6. **Use Binary Serialization**: For performance-critical paths
7. **Monitor Storage Size**: Implement cleanup strategies
8. **Avoid Deep Nesting**: Keep data structures flat when possible

## Debugging

### Enable Debug Logging
```typescript
const storage = new StorageContainer({
    backend: new MemoryBackend(),
    serializer: new JSONSerializer(),
    debug: true,
    logger: console
});
```

### Get Storage Statistics
```typescript
const stats = await storage.getStats();
console.log('Storage Stats:', {
    items: stats.itemCount,
    size: `${(stats.totalSize / 1024 / 1024).toFixed(2)} MB`,
    backend: stats.backendType,
    hitRate: stats.hitRate
});
```

### Inspect Stored Data
```typescript
const keys = await storage.keys();
for (const key of keys) {
    const value = await storage.get(key);
    console.log(`${key}:`, value);
}
```

## Common Pitfalls

1. **Not Initializing**: Always call `await storage.initialize()` first
2. **Forgetting Async/Await**: All storage operations are asynchronous
3. **Circular References**: Ensure data is serializable (no circular refs)
4. **Large Objects**: Consider chunking or streaming for very large data
5. **Memory Leaks**: Clear cache periodically with `clear()` or TTL
6. **Type Mismatches**: Use TypeScript generics for type safety
7. **Missing Error Handling**: Always catch storage errors
8. **Quota Limits**: Monitor and handle quota exceeded errors

## Migration Guide

### Migrating from localStorage
```typescript
// Old: localStorage
localStorage.setItem('key', JSON.stringify(data));
const data = JSON.parse(localStorage.getItem('key'));

// New: General Storage
await storage.set('key', data);
const data = await storage.get('key');
```

### Migrating from Custom File Storage
```typescript
// Old: fs module
const fs = require('fs').promises;
await fs.writeFile('data.json', JSON.stringify(data));
const data = JSON.parse(await fs.readFile('data.json', 'utf8'));

// New: General Storage
const storage = new StorageContainer({
    backend: new FileSystemBackend({ basePath: './' })
});
await storage.set('data', data);
const data = await storage.get('data');
```

## Testing

### Mock Storage for Tests
```typescript
import { MemoryBackend } from '@samsara/storage';

// Test setup
let storage: IStorageContainer;

beforeEach(async () => {
    storage = new StorageContainer({
        backend: new MemoryBackend(),
        serializer: new JSONSerializer()
    });
    await storage.initialize();
});

afterEach(async () => {
    await storage.clear();
});

// Test
test('stores and retrieves data', async () => {
    await storage.set('test', { value: 123 });
    const result = await storage.get('test');
    expect(result).toEqual({ value: 123 });
});
```

## Resources

- [Full Design Document](./GeneralStorage_Design.md)
- [API Reference](./api/README.md)
- [Examples Repository](../examples/)
- [Performance Benchmarks](./benchmarks/README.md)
- [Contributing Guide](../CONTRIBUTING.md)
