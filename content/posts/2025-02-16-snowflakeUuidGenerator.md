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

**Timestamp (41 bits)**: Represents the timestamp in milliseconds since a custom epoch.

**Datacenter ID (5 bits)**: Identifies the datacenter or region.

**Worker ID (5 bits)**: Identifies the machine or node within the datacenter.

**Sequence Number (12 bits)**: A counter that increments per ID generation within the same millisecond.

**Sign Bit (1 bit)**: Reserved for future use (typically unused and set to 0).

This design allows Snowflake IDs to be unique, time-ordered, and generated in a distributed fashion without coordination.

## Benefits of Snowflake ID

**Uniqueness**: Guarantees uniqueness without relying on centralized databases.

**Scalability**: Supports high throughput ID generation in distributed systems.

**Sortability**: IDs are time-ordered, which improves database indexing and querying performance.

**Efficiency**: Generates IDs with minimal latency (~1 microsecond per ID in most implementations).

## Example Implementation

A simple Python implementation of the Snowflake ID Generator:

```cs
using System;
using System.Threading;

public class SnowflakeIDGenerator
{
    private static readonly object _lock = new object();
    private const long Epoch = 1640995200000L;
    private const int DatacenterIdBits = 5;
    private const int WorkerIdBits = 5;
    private const int SequenceBits = 12;
    private const long MaxDatacenterId = (1L << DatacenterIdBits) - 1;
    private const long MaxWorkerId = (1L << WorkerIdBits) - 1;
    private const long MaxSequence = (1L << SequenceBits) - 1;
    private readonly long _datacenterId;
    private readonly long _workerId;
    private long _lastTimestamp = -1L;
    private long _sequence = 0L;

    public SnowflakeIDGenerator(long datacenterId, long workerId)
    {
        if (datacenterId > MaxDatacenterId || datacenterId < 0)
            throw new ArgumentException("Datacenter ID out of range");
        if (workerId > MaxWorkerId || workerId < 0)
            throw new ArgumentException("Worker ID out of range");

        _datacenterId = datacenterId;
        _workerId = workerId;
    }

    private long CurrentTimestamp() => DateTimeOffset.UtcNow.ToUnixTimeMilliseconds();

    public long GenerateId()
    {
        lock (_lock)
        {
            long timestamp = CurrentTimestamp();
            if (timestamp == _lastTimestamp)
            {
                _sequence = (_sequence + 1) & MaxSequence;
                if (_sequence == 0)
                {
                    while ((timestamp = CurrentTimestamp()) <= _lastTimestamp) { }
                }
            }
            else
            {
                _sequence = 0;
            }

            _lastTimestamp = timestamp;
            return ((timestamp - Epoch) << (DatacenterIdBits + WorkerIdBits + SequenceBits)) |
                   (_datacenterId << (WorkerIdBits + SequenceBits)) |
                   (_workerId << SequenceBits) |
                   _sequence;
        }
    }
}

// Example usage:
class Program
{
    static void Main()
    {
        var generator = new SnowflakeIDGenerator(1, 1);
        Console.WriteLine(generator.GenerateId());
    }
}
```

## Use Cases

**Distributed Databases**: Generating unique primary keys for NoSQL and relational databases.

**Messaging Systems**: Assigning unique message IDs in distributed messaging applications.

**Microservices**: Ensuring unique request and transaction identifiers across services.

**Logging & Analytics**: Generating IDs for logs, traces, and events.

## Conclusion

The Snowflake Unique ID Generator is a powerful approach for distributed ID generation, ensuring uniqueness, scalability, and efficiency. By leveraging time-based sorting and decentralization, it enables high-performance applications that require large-scale, unique ID creation.