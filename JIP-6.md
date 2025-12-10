# JIP-6: Structured execution logs

A common format for PVM execution logging. The goal is to easily compare execution traces between implementations and identify divergence points.

## Format specification

The trace format captures the state of the PVM during execution. Most fields are optional to allow implementations to balance performance and debugging ability.

Data is encoded using the standard JAM codec, also called GP codec. All numbers use variable-length encoding as defined in the Gray Paper.

The format is binary. Alternatively, the data can be base64-encoded and delimited by newlines.

## Trace versions

Two trace versions are defined:

- **Full**: Full PVM tracing of each instruction. May include extra entries for `ecalli` resuming.
- **Ecalli**: Host calls stop tracing. One log entry is created at `ecalli` instruction and one entry when execution resumes.

## Types

### Trace

Top-level trace collection with the following fields:

- `format`: Format type (u32). Either `full` (0) or `ecalli` (1).
- `implementation`: String identifying the JAM implementation that produced the trace.
- `codehash`: Optional 32-byte hash of the executed code.
- `initial`: Optional initial PVM state before the first instruction.
- `entries`: List of log entries with PVM state after each instruction.
- `status`: Final PVM status. Either `Panic` (0) or `Halt` (1).

### Log

Single log entry showing PVM state after one instruction:

- `pc`: Program counter (u32).
- `gas`: Optional gas value after executing the instruction at pc (u64).
- `registers`: Optional register state after executing the instruction. Array of 13 u64 values. Can be omitted if registers do not change.
- `memory`: List of memory chunks read or written by the instruction.

For host calls, one log entry is created for the `ecalli` instruction to inspect host call arguments in registers. A second entry is added by the host environment with changes to the VM state.

Host call arguments may be provided as memory writes in the initial log entry.

### Chunk

Memory read or write data:

- `access`: Access type. Either `Read` (0) or `Write` (1).
- `address`: Memory location (u32).
- `data`: Either length of accessed memory (u32) or the actual memory content as bytes.

## ASN.1 notation

```asn1
Trace ::= SEQUENCE {
    format Format,
    codehash OCTET STRING (SIZE(32)) OPTIONAL,
    initial Log OPTIONAL,
    entries SEQUENCE OF Log,
    status Status,
    implementation UTF8String
}

Format ::= INTEGER {
    full(0),
    ecalli(1)
}

Status ::= INTEGER {
    panic(0),
    halt(1)
}

Log ::= SEQUENCE {
    pc INTEGER,
    gas INTEGER OPTIONAL,
    registers SEQUENCE (SIZE(13)) OF INTEGER OPTIONAL,
    memory SEQUENCE OF Chunk
}

Chunk ::= SEQUENCE {
    access Access,
    address INTEGER,
    data MemoryData
}

MemoryData ::= CHOICE {
    length INTEGER,
    data OCTET STRING
}

Access ::= INTEGER {
    read(0),
    write(1)
}
```

## Examples

Typescript parser & printer: https://github.com/tomusdrw/jip-logger

<!-- TODO: Add example trace for simple program -->
<!-- TODO: Add example with host call -->
<!-- TODO: Add example showing divergence detection -->
