+++
date = '2025-02-16'
draft = false
title = 'Snowflake Unique ID Generator'
+++

# Snowflake Unique ID Generator

## Introduction

A Snowflake Unique ID Generator is a distributed, high-performance unique identifier generation system designed to create unique IDs at scale. Originally developed by Twitter, the Snowflake ID algorithm ensures globally unique, time-ordered, and efficiently sortable IDs without requiring a centralized database.

## Structure of a Snowflake ID

A Snowflake ID is typically a 64-bit integer composed of multiple segments that encode metadata. The structure is as follows:

* **Timestamp (41 bits)**: Represents the timestamp in milliseconds since a custom epoch.
* **Datacenter ID (5 bits)**: Identifies the datacenter or region.
* **Worker ID (5 bits)**: Identifies the machine or node within the datacenter.
* **Sequence Number (12 bits)**: A counter that increments per ID generation within the same millisecond.
* **Sign Bit (1 bit)**: Reserved for future use (typically unused and set to 0).

This design allows Snowflake IDs to be unique, time-ordered, and generated in a distributed fashion without coordination.

## Benefits of Snowflake ID

* **Uniqueness**: Guarantees uniqueness without relying on centralized databases.
* **Scalability**: Supports high throughput ID generation in distributed systems.
* **Sortability**: IDs are time-ordered, which improves database indexing and querying performance.
* **Efficiency**: Generates IDs with minimal latency (~1 microsecond per ID in most implementations).

## Example Implementation

A simple Python implementation of the Snowflake ID Generator:

```cs
public class SnowflakeIdGenerator
{
    private const long Twepoch = 1288834974657L; // Custom epoch timestamp
    private const int MachineIdBits = 5;
    private const int DataCenterIdBits = 5;
    private const int SequenceBits = 12;

    private const long MaxMachineId = -1L ^ (-1L << MachineIdBits);
    private const long MaxDataCenterId = -1L ^ (-1L << DataCenterIdBits);
    private const long MaxSequence = -1L ^ (-1L << SequenceBits);

    private const int MachineIdShift = SequenceBits;
    private const int DataCenterIdShift = SequenceBits + MachineIdBits;
    private const int TimestampLeftShift = SequenceBits + MachineIdBits + DataCenterIdBits;

    private readonly long _machineId;
    private readonly long _dataCenterId;
    private long _lastTimestamp = -1L;
    private long _sequence = 0L;

    private readonly object _lock = new();

    public SnowflakeIdGenerator(long machineId, long dataCenterId)
    {
        if (machineId > MaxMachineId || machineId < 0)
            throw new ArgumentException($"Machine ID must be between 0 and {MaxMachineId}");

        if (dataCenterId > MaxDataCenterId || dataCenterId < 0)
            throw new ArgumentException($"Datacenter ID must be between 0 and {MaxDataCenterId}");

        _machineId = machineId;
        _dataCenterId = dataCenterId;
    }

    public long NextId()
    {
        lock (_lock)
        {
            long timestamp = GetCurrentTimestamp();

            if (timestamp < _lastTimestamp)
            {
                throw new InvalidOperationException("Clock moved backwards. Refusing to generate ID.");
            }

            if (timestamp == _lastTimestamp)
            {
                _sequence = (_sequence + 1) & MaxSequence;
                if (_sequence == 0) timestamp = WaitForNextMillis(_lastTimestamp);
            }
            else
            {
                _sequence = 0;
            }

            _lastTimestamp = timestamp;

            return ((timestamp - Twepoch) << TimestampLeftShift)
                   | (_dataCenterId << DataCenterIdShift)
                   | (_machineId << MachineIdShift)
                   | _sequence;
        }
    }

    private long GetCurrentTimestamp()
    {
        return DateTimeOffset.UtcNow.ToUnixTimeMilliseconds();
    }

    private long WaitForNextMillis(long lastTimestamp)
    {
        long timestamp = GetCurrentTimestamp();
        while (timestamp <= lastTimestamp)
        {
            timestamp = GetCurrentTimestamp();
        }
        return timestamp;
    }
}

// Example usage:
class Program
{
    static void Main()
    {
        var generator = new SnowflakeIdGenerator(machineId: 1, dataCenterId: 1);
        long uniqueId = generator.NextId();
        Console.WriteLine($"Generated Snowflake ID: {uniqueId}");
    }
}
```

## Performance Benchmarking

```cs
using System.Collections.Concurrent;
using System.Diagnostics;
using IdGenerator;

int threadCount = 50;  // Number of parallel threads
int idCountPerThread = 1000000; // Number of IDs per thread
var generator = new SnowflakeIdGenerator(machineId: 1, dataCenterId: 1);

var idSet = new ConcurrentDictionary<long, bool>(); // Store IDs to check for duplicates
int duplicateCount = 0;
var stopwatch = Stopwatch.StartNew();

Parallel.For(0, threadCount, _ =>
{
    for (int i = 0; i < idCountPerThread; i++)
    {
        long id = generator.NextId();

        // If ID already exists, it's a collision
        if (!idSet.TryAdd(id, true))
        {
            Console.WriteLine($"⚠️ Duplicate ID detected: {id}");
            lock (idSet)
            {
                duplicateCount++;
            }
        }
    }
});

stopwatch.Stop();
long totalGenerated = threadCount * idCountPerThread;

Console.WriteLine($"\n📌 Test Summary:");
Console.WriteLine($"✅ Total IDs Generated: {totalGenerated}");
Console.WriteLine($"❌ Duplicates Found: {duplicateCount}");
Console.WriteLine($"⏱️ Execution Time: {stopwatch.ElapsedMilliseconds} ms");
Console.WriteLine($"⚡ Speed: {totalGenerated / (stopwatch.ElapsedMilliseconds / 1000.0)} IDs/sec");

```

### Expected Results

📌 Test Summary:
✅ Total IDs Generated: 50000000
❌ Duplicates Found: 0
⏱️ Execution Time: 21469 ms
⚡ Speed: 2328939.400996786 IDs/sec

## Use Cases

* **Distributed Databases**: Generating unique primary keys for NoSQL and relational databases.
* **Messaging Systems**: Assigning unique message IDs in distributed messaging applications.
* **Microservices**: Ensuring unique request and transaction identifiers across services.
* **Logging & Analytics**: Generating IDs for logs, traces, and events.

## Conclusion

The Snowflake Unique ID Generator is a powerful approach for distributed ID generation, ensuring uniqueness, scalability, and efficiency. By leveraging time-based sorting and decentralization, it enables high-performance applications that require large-scale, unique ID creation.
