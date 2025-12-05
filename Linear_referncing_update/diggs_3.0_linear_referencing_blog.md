# Linear Referencing in DIGGS 3.0: An Updated Approach to Positioning Data Along Boreholes and Other Linear Features

*Daniel Ponti | DIGGS Technical Committee*

---

## Introduction: Linear Referencing and DIGGS

In geotechnical and geoenvironmental investigations, we constantly need to record *where* something was collected, observed or measured. For features that follow a linear path—boreholes, piezometers, cone penetration soundings, piles, geophysical tracklines, transects and trial pits—the most natural way to specify position is to measure distance along that path. This approach is called **linear referencing**.

 > **Why Linear Referencing Is Important**
 >
 >When a geologist logging a borehole describes a soil layer as "12-18 ft depth; stiff gray clay", they are using linear referencing—measured depth along the borehole path—because that's the natural coordinate system for the work. Any transfer standard must therefore support linearly referenced coordinates.
 
 However, in the example above, the coordinates defining the position of a linearly referenced soil layer (12 and 18 ft. respectively) are one-dimensional (hence *linearly* referenced) and by themselves do not provide the true spatial location of the layer. To determine that, we must also know the coordinates of the borehole path in a spatial (typically 3D) coordinate reference system. This information thus allows the true spatial position of the layer to be interpolated from the linearly referenced coordinates.

DIGGS supports both absolute spatial positioning and linearly referenced positioning by providing a rigorous framework for defining linear spatial reference systems and encoding location coordinates in a standardized, machine-readable format that allows processing applications to understand and convert between linear referenced coordinates and the true spatial coordinates of samples, observations and measurements.

## DIGGS 3.0: A Significant Update

Previous DIGGS versions supported linear referencing by adopting a subset of the GML 3.3 Linear Referencing schema for defining a linear spatial reference system. DIGGS 3.0 introduces a redesigned linear referencing framework based on the latest **ISO 19148:2021 (Linear referencing)** standard. While implementation of the new approach in DIGGS instances is very similar to how linear spatial reference systems were defined previously, the DIGGS 3.0 update is optimized specifically for geotechnical and geoenvironmental applications and offers enhancements over previous versions .

### Why the Change?

The GML 3.3 Linear Referencing schema followed the 2012 version of ISO 19148 and was designed primarily for transportation applications. As implemented in DIGGS, the GML schema was subject to encoding variations  that could result in inconsistent encoding, thus hampering interoperability. More importantly, the 2021 revision of ISO 19148 clarified and modernized linear referencing concepts, providing an opportunity to build a cleaner, more focused implementation.

**Key Benefits:**

- **✓ ISO 19148:2021 Compliance**: Full alignment with the latest international standard for linear referencing.
- **✓ Better Constrained Schema**:  Added geotechnical-specific enumerations to elements, inproving validation and interoperability.
-  **✓ A New Linear Referencing Method Dictionary**: Pre-defined, standard referencing methods enhance interoperability and adoption, reduces redundancy, and  provides for more compact encoding.
- **✓ Enhanced Functionality**: Support for handling stretched or compressed core samples, the use of referents, and 2D positioning with lateral offsets for transects, tracklines and trial pits.
- **✓ Future Proofing**: Full support for other linear features, such as tunnels or pipelines, that might be introduced in future versions of DIGGS.

## Core Concepts: The Building Blocks

DIGGS 3.0 linear referencing is built on three fundamental components defined by ISO 19148:

1. **Linear Element (diggs:LinearExtent object)**: The geometric feature along which measurements are made—the physical path of the borehole, trackline, or transect.

2. **Linear Referencing Method (diggs:LRMethod object)**: The procedure or algorithm for determining positions—how measurements are calculated and what units are used.

3. **Linear Spatial Reference System (diggs:LinearSRS)**: The complete coordinate system created by combining a Linear Element with a Linear Referencing Method.

![Figure 1: Linear Referencing Components](figure1_lr_components.svg)
*Figure 1: The three core components of DIGGS linear referencing combine to create a complete coordinate reference system*

## Understanding Linear Elements

A **LinearExtent** defines the geometric path along which measurements are made. For a borehole, this is typically a 3D line from the collar (top) to the bottom of the hole. The geometry is stored as an ordered list of coordinates.

Here's how a simple borehole's centerline would be defined in DIGGS:

```xml
<centerLine>
    <LinearExtent gml:id="cl-B-001-0-20"
                  srsDimension="3" 
                  srsName="https://www.opengis.net/def/crs-compound?1=http://www.opengis.net/def/crs/EPSG/0/4326%262=http://www.opengis.net/def/crs/EPSG/0/5702"> 
                  
        <!-- srsName URL defines compound CRS with horizontal lat/lon (WGS84) and NAVD88 height in feet-->
        <gml:posList>39.474660 -81.796858 818.6 39.474660 -81.796858 777.6</gml:posList>
        <datumReference>groundSurface</datumReference>
    </LinearExtent>
</centerLine>
```

The gml:posList element holds 2 3D coordinate tuples for the path, in the spatial coordinate reference system defined by the srsName attribute. These coordinates describe a vertical borehole at a specific latitude/longitude, extending from elevation 818.6 feet (ground surface) down to 777.6 feet (41 feet total depth). The `datumReference` value derives from an enumerated list and indicates that the origin of the path is at the ground surface.

More complex paths that would describe an inclined or deviated borehole can be defined by simply altering and/or adding more coordinate tuples to define the path in 3D space. No other properties are necessary for defining complex paths.

![Figure 2: LinearExtent Geometry](figure2_linearextent_geometry.svg)
*Figure 2: LinearExtent defines the geometric path of the borehole with ordered coordinates from top to bottom*

## Linear Referencing Methods

An **LRMethod** object defines *how* positions are determined along the linear element. DIGGS 3.0 supports three types per ISO 19148:

| Method Type | Description | Example Use |
|-------------|-------------|-------------|
| **Absolute** | Distance from the linear element origin (typically first coordinate) | Measured depth from top of hole |
| **Relative** | Distance from a specified referent point | Depth below ground surface where, for example,linear element origin is referenced to the drill rig's rotary table|
| **Interpolative** | Position interpolated between control points | Observations on core samples where the cores are stretched or compressed relative to the cored interval|

For the above example, the LRMethod object for measured depth down the borehole in feet would be encoded as follows:

```xml
<LRMethod gml:id="md_ft">
    <gml:description>Absolute measured depth from top of borehole or sounding in feet. Measurements are cumulative 3D distance along the borehole/sounding centerline from the first coordinate (typically top of hole or rig datum) proceeding downward through the hole. Suitable for vertical, slanted, and deviated boreholes and soundings.</gml:description>
    <gml:identifier codeSpace="DIGGS">#md_ft</gml:identifier>
    <gml:name>Absolute Measured Depth (ft)</gml:name>
    <type>absolute</type>
    <units>ft</units>
    <positiveDirection>downward</positiveDirection>
    <anchorDefinition>Measurement in feet from the first coordinate of the centerline LinearExtent, which represents "top of hole" (or rig datum such as kelly bushing) with positive downward along the borehole path</anchorDefinition>
    <requiresCalibration>false</requiresCalibration>
</LRMethod>
```

### The LRMethod Dictionary: A Time-Saver

Rather than defining LRMethods in every data file, DIGGS 3.0 provides a **standard dictionary** of common methods at:

**(https://diggsml.org/def/crs/DIGGS/0.1/lrm.xml}**

This dictionary includes 16 pre-defined methods covering:

- Borehole measured depth (absolute and relative, feet and meters)
- Trackline/transect distances (absolute and relative, feet and meters)
- Core sample positioning (top-down, bottom-up, interpolative)
- CPT soundings, pipelines, tunnels, and elevation-based systems

Users can reference these by URL in their LinearSRS definitions, eliminating the need to encode the same LRMethod objects repeatedly. This promotes consistency, interoperability, and reduces file size.

## Creating a Linear SRS

A **LinearSRS** object combines a LinearExtent with an LRMethod to create a complete coordinate reference system. This is the key object that gets referenced (with the `srsName` attribute) when using linear referenced coordinates.

### Example 1: Absolute Measured Depth

Here's how to define a linear reference system for measuring absolute depth from top of hole in the above example:

```xml
<linearReferencing>
    <LinearSRS gml:id="lsr-B-001-0-20">
        <gml:identifier codeSpace="ODOT">lsr-B-001-0-20</gml:identifier>
        <gml:name>Depth reference system for B-001-0-20</gml:name>
        
        <!-- Reference to the borehole's centerline geometry (previously defined LinearExtent in the instance) -->
        <linearElement xlink:href="#cl-B-001-0-20"/>
        
        <!-- Reference to the DIGGS standard LRMethod for measured depth down hole in feet -->
        <lrm xlink:href="https://diggsml.org/def/crs/DIGGS/0.1/lrm.xml#md_ft"/>
    </LinearSRS>
</linearReferencing>
```

Now, anywhere in your DIGGS file, you can reference this coordinate system:

```xml
<!-- Backfill from 0 to 41 feet depth -->
<LinearExtent srsDimension="1" srsName="#lsr-B-001-0-20">
    <gml:posList>0.00 41.00</gml:posList>
</LinearExtent>

<!-- Water level at 6 feet depth below top of hole -->
<PointLocation srsDimension="1" srsName="#lsr-B-001-0-20">
    <gml:pos>9.00</gml:pos>
</PointLocation>
```

![Figure 3: Absolute Linear Referencing](figure3_absolute_referencing.svg)
*Figure 3: Absolute linear referencing measures all positions from a fixed origin (top of hole)*

### Example 2: Relative Referencing with Referents

Sometimes you need to measure from a point other than the origin. For example, after installing casing in a vault below ground surface, you might want to reference water levels from the top of casing rather than from the original ground surface. This is where **referents** come in.

A **referent** is a reference point along the linear feature. Common referents include ground surface, top of casing, seafloor, kelly bushing, or any other significant datum point. When using a *relative* LRMethod, you specify which referent serves as the origin (zero point) for measurements.

```xml
<linearReferencing>
    <LinearSRS gml:id="lsr-B-001-0-20_casingTop">
        <gml:name>Depth from top of casing in B-001-0-20</gml:name>
        <linearElement xlink:href="#cl-B-001-0-20"/>
        
        <lrm xlink:href="https://diggsml.org/def/crs/DIGGS/0.1/lrm.xml#relative_md_ft"/>
       
        <!-- Specify the referent (origin) for this system -->
        <referent>
            <Referent>
                <referentType>topOfCasing</referentType>
                <position>
                    <PointLocation srsDimension="3" srsName="...Spatial CRS URL...">
                        <gml:pos>39.474660 -81.796858 816.6</gml:pos>
                    </PointLocation>
                </position>
                <!-- Location along the borehole (2 ft from top) -->
                <distanceAlong uom="ft">2</distanceAlong>
            </Referent>
        </referent>
    </LinearSRS>
</linearReferencing>
```

Now you can measure a water level relative to the top of casing:

```xml
<!-- Water level 7 feet below top of casing -->
<PointLocation srsDimension="1" srsName="#lsr-B-001-0-20_casingTop">
    <gml:pos>7.00</gml:pos>
</PointLocation>
```

![Figure 4: Relative Linear Referencing](figure4_relative_referencing.svg)
*Figure 4: Relative linear referencing uses a referent (top of casing) as the origin. The same physical location has different coordinate values in each system.*

### Real-World Application: Multiple Reference Systems

In the provided example, Borehole B-001-0-20 has **two LinearSRS definitions**:

1. **lsr-B-001-0-20**: Absolute measured depth from top of hole (used for drilling operations, backfill intervals, construction methods)
2. **lsr-B-001-0-20_casingTop**: Relative depth from top of casing (used for post-installation water level monitoring)

The initial water strike at 9 feet depth (during drilling) references the absolute system. A follow-up reading one day later, after casing installation, references the relative system at 7 feet below top of casing. Both describe the same physical water level, but use different coordinate systems appropriate to when and how the measurement was made.

> **Important:** Having multiple LinearSRS definitions for a single borehole is perfectly valid and often necessary. Each represents a different way of measuring position along the same physical feature, appropriate for different phases of work or different measurement conventions.

## Advanced Features for DIGGS 3.0 Linear Referencing

### Anchor Points for Core Stretch/Compression

When core samples are recovered, the physical core length may differ from the cored interval due to expansion (common in clay-rich soils) or compression (from sampling disturbance or dessication). **Anchor points** provide calibration points to adjust positions between the physical core and the borehole depth system.

For example, if you core from 10-15 feet (5 feet drilled interval) but recover 5.5 feet of core (10% expansion), you can use an *interpolative* LRMethod with anchor points at the top and bottom of the core. This scales positions proportionally so that observations on the physical core can be accurately tied back to depth in the borehole. Anchor points are defined in the LinearExtent object defined for the core sample.

### 2D Linear Referencing with Lateral Offsets

For features like transects, trial pits, or geophysical tracklines, you may need to record positions that are offset from the centerline. DIGGS 3.0 supports **2D linear referencing** where coordinates include both distance along and lateral offset.

An LRMethod that includes the  `OffsetParameters` object becomes a 2D method. Positions are then encoded as `<gml:pos>distance offset</gml:pos>`, where the first value is distance along the centerline LinearExtent and the second is the offset (by default positive to the right, negative to the left when facing the positive centerline direction). The `offsetType` property of the `OffsetParameters` object defines how the offset is measured from the centerline (perpendicular, constantBearing, or radial)

## Migration from DIGGS 2.6

If you're currently using DIGGS 2.6 with the GML 3.3/lr-based linear referencing, here's what you need to know about migrating your instances to DIGGS 3.0:

### Key Changes

- **Element names**: `LinearSpatialReferenceSystem` becomes `LinearSRS` (legacy name using the GML 3.3 encoding will still validate under 3.0, but is deprecated)
- **Namespace**: Linear referencing elements that used the GML glr: namespace are now in the DIGGS namespace (`diggs:`).

The conceptual model remains the same—you're still defining coordinate systems for linear features—but the encoding is cleaner and better aligned with current standards.

## Benefits of the New Approach

**Standards Compliance**: Full alignment with ISO 19148:2021, ensuring interoperability with other geospatial systems and future-proofing your data.

**Clarity & Simplicity**: The schema directly reflects geotechnical workflows with clearly defined enumerated values for defining linear reference system properties.

**Enhanced Features**: Support referents, stretched/compressed core samples, and lateral offset positions.
**Reusability**: Standard LRMethod dictionary eliminates redundant encoding. Reference common methods by URL instead of duplicating definitions.

## Getting Started

Ready to implement DIGGS 3.0 linear referencing in your projects? Here are your next steps:

1. **Review the Schema**: Until 3.0 is officially released, the linear referencing schema is available on GitHub at https://github.com/DIGGSml/schema-dev/tree/3.0.
2. **Explore the Dictionary**: Browse the standard [LRMethod dictionary](https://diggsml.org/def/crs/DIGGS/0.1/lrm.xml)
3. **Review Examples**: Review [this example instance](https://github.com/DIGGSml/schema-dev/blob/3.0/Instances/DocExample3.0.xml) in the DIGGS repository to see linear referencing in action
4. **Join the DIGGS Effort**
    - **Github**: Contribute to our repositories at https://github.com/DIGGSml/">https://github.com/DIGGSml
    - **Monthly Meetings: Join our monthly onlinediscussions where we tackle issues like this one. Contact 
            <a href="mailto:acadden@schnabel-eng.com">Allen Cadden</a> or <a href="mailto:rcutts@schnabel-eng.com">Ross Cutts</a> to receive
            meeting invites.
    - **Feedback**: Share your experiences implementing these changes
  
## Conclusion

Linear referencing is fundamental to how we work with geotechnical data. The DIGGS 3.0 approach provides a robust, standards-compliant framework that directly serves the needs of our community and will grow with it. By adopting ISO 19148:2021 principles and focusing on geotechnical workflows, we've created a system that's both powerful and intuitive.

Whether you're encoding borehole logs, CPT soundings, geophysical surveys, or any other linear sampling feature, DIGGS 3.0 gives you the tools to precisely, consistently, and interoperably describe where your data comes from. And with the standard LRMethod dictionary, implementing linear referencing in your DIGGS files is easier than ever.

We encourage you to explore the new linear referencing schema, experiment with examples, and share your feedback. Together, we're building the future of geotechnical data exchange.
