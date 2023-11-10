# Flex Particles Phase

## Particles
Particles are the fundamental building-block in Flex. For each particle the basic dynamic quantities of position, velocity, and inverse mass are stored. The inverse mass is stored alongside the position, in the format [x, y, z, 1/m]. In addition to these properties, Flex also stores a per-particle phase value, which controls the particle behavior (see the Phase section for more information). Below is a simple example showing the layout for particle data, and how to set it onto the API.:

```C++
NvFlexBuffer* particleBuffer = NvFlexAllocBuffer(library, n, sizeof(Vec4), eNvFlexBufferHost);
NvFlexBuffer* velocityBuffer = NvFlexAllocBuffer(library, n, sizeof(Vec4), eNvFlexBufferHost);
NvFlexBuffer* phaseBuffer = NvFlexAllocBuffer(library, n, sizeof(int), eNvFlexBufferHost);

// map buffers for reading / writing
float4* particles = (float4*)NvFlexMap(particlesBuffer, eNvFlexMapWait);
float3* velocities = (float3*)NvFlexMap(velocityBuffer, eNvFlexMapWait);
int* phases = (int*)NvFlexMap(phaseBuffer, eFlexMapWait);

// spawn particles
for (int i=0; i < n; ++i)
{
        particles[i] = RandomSpawnPosition();
        velocities[i] = RandomSpawnVelocity();
        phases[i] = NvFlexMakePhase(0, eNvFlexPhaseSelfCollide);
}

// unmap buffers
NvFlexUnmap(particleBuffer);
NvFlexUnmap(velocityBuffer);
NvFlexUnmap(phaseBuffer);

// write to device (async)
NvFlexSetParticles(solver, particleBuffer, NULL);
NvFlexSetVelocities(solver, velocityBuffer, NULL);
NvFlexSetPhases(solver, phaseBuffer, NULL);
```

## Phase

Each particle in Flex has an associated phase, this is a 32 bit integer that controls the behavior of the particle in the simulation. A phase value consists of a group identifier, and particle flags, as described below:

- **eNvFlexPhaseGroupMask** - The particle group is an arbitrary positive integer, stored in the lower 24 bits of the particle phase. Groups can be used to organize related particles and to control collisions between them. The collision rules for groups are as follows: particles of different groups will always collide with each other, but by default particles within the same group will not collide. This is useful, for example, if you have a rigid body made up of many particles and do not want the internal particles of the body to collide with each other. In this case all particles belonging to the rigid would have the same group value.
- **eFlexPhaseSelfCollide** - To enable particles of the same group to collide with each other this flag should be specified on the particle phase. For example, for a piece of cloth that deforms and needs to collide with itself, each particle in the cloth could have the same group and have this flag specified.
- **eNvFlexPhaseFluid** - When this flag is set the particle will also generate fluid density constraints, cohesion, and surface tension effects with other fluid particles. Note that generally fluids should have both the fluid and self-collide flag set, otherwise they will only interact with particles of other groups.

The API comes with a helper function, **NvFlexMakePhase()**, that generates phase values given a group and flag set. The example below shows how to set up phases for the most common cases:

```C++
// create a fluid phase that self collides and generates density constraints
int fluidPhase = NvFlexMakePhase(0, eNvFlexPhaseSelfCollide | eNvFlexPhaseFluid);

// create a rigid phase that collides against other groups, but does not self-collide
int rigidPhase = NvFlexMakePhase(1, 0);

// create a cloth phase that collides against other groups and itself
int clothPhase = NvFlexMakePhase(2, eNvFlexPhaseSelfCollide);
```

## Test

### eNvFlexPhaseSelfCollide | eNvFlexPhaseSelfCollideFilter

| Instances | group | channel |
|-----------|-------|---------|
| Instance0 | 0     | 0       |
| Instance1 | 0     | 1       |
| Instance2 | 0     | 2       |

Collide with each group

| Instances | group | channel |
|-----------|-------|---------|
| Instance0 | 0     | 0       |
| Instance1 | 0     | 0       |
| Instance2 | 0     | 0       |

Collide with each group

| Instances | group | channel |
|-----------|-------|---------|
| Instance0 | 0     | 0       |
| Instance1 | 1     | 0       |
| Instance2 | 2     | 0       |

Collide with each group

| Instances | group | channel |
|-----------|-------|---------|
| Instance0 | 0     | 0       |
| Instance1 | 1     | 1       |
| Instance2 | 2     | 2       |

Collide with each group
### eNvFlexPhaseSelfCollideFilter

| Instances | group | channel |
|-----------|-------|---------|
| Instance0 | 0     | 0       |
| Instance1 | 0     | 1       |
| Instance2 | 0     | 2       |

no collide with each other

| Instances | group | channel |
|-----------|-------|---------|
| Instance0 | 0     | 0       |
| Instance1 | 0     | 0       |
| Instance2 | 0     | 0       |

no collide with each other

| Instances | group | channel |
|-----------|-------|---------|
| Instance0 | 0     | 0       |
| Instance1 | 1     | 0       |
| Instance2 | 2     | 0       |

Collide with each group

| Instances | group | channel |
|-----------|-------|---------|
| Instance0 | 0     | 0       |
| Instance1 | 1     | 1       |
| Instance2 | 2     | 2       |

Collide with each group



