# Linear Referencing in DIGGS 3.0:  Improvements for Positioning Data Along Boreholes and Other Linear Features

*Daniel Ponti | DIGGS Technical Committee*

---

## Introduction: Linear Referencing and DIGGS

In geotechnical and geoenvironmental investigations, we constantly need to record *where* something was collected, observed or measured. For features that follow a linear path—boreholes, wells, cone penetration soundings, piles, geophysical tracklines, transects and trial pits—the most natural way to specify position is to measure distance along that path. This approach is called **linear referencing**.

> **Why Linear Referencing Is Important**
>
> When an engineering geologist logging a borehole describes a soil layer as "12-18 ft depth; stiff gray clay", they are using linear referencing—measured depth along the borehole path—because that's the natural coordinate system for the work. Any transfer standard must therefore support linearly referenced coordinates.

However, the coordinates defining the position of a linearly referenced soil layer (12 and 18 ft. in the example above   ) are one-dimensional (hence *linearly* referenced) and by themselves do not provide the true spatial location of the layer. To determine that, we must also know the coordinates of the borehole path in a spatial (typically 3D) coordinate reference system. This information thus allows the true spatial position of the layer to be interpolated from the linearly referenced coordinates.

DIGGS supports both absolute spatial positioning and linearly referenced positioning by providing a rigorous framework for defining linear spatial reference systems and encoding location coordinates in a standardized, machine-readable format that allows processing applications to understand and convert between linear referenced coordinates and the true spatial coordinates of samples, observations and measurements.

## Updates to Linear Referencing in DIGGS 3.0

Previous DIGGS versions supported linear referencing by directly using a limited subset of the GML 3.3 Linear Referencing schema for defining a linear spatial reference system. DIGGS 3.0 introduces an updated linear referencing framework based on the latest **ISO 19148:2021 (Linear referencing)** standard. It is optimized specifically for geotechnical and geoenvironmental applications, and offers enhancements over previous versions.

### Why the Change?

The GML 3.3 Linear Referencing schema followed the 2012 version of ISO 19148 and was designed primarily for transportation applications. As implemented in DIGGS, the GML schema was subject to inconsistent encoding of key properties, thus hampering interoperability. More importantly, the 2021 revision of ISO 19148 clarified and modernized linear referencing concepts, providing an opportunity to build a cleaner, more focused implementation.

**Key Benefits:**

- **✓ ISO 19148:2021 Compliance**: Full alignment with the latest international standard for linear referencing.
- **✓ Better Constrained Schema**: Added geotechnical-specific enumerations to properties, improving validation and interoperability.
- **✓ A New Linear Referencing Method Dictionary**: Pre-defined, standard referencing methods enhance interoperability and adoption, reduce redundancy, and provide for more compact encoding.
- **✓ Enhanced Functionality**: Support for handling stretched or compressed core samples, the use of referents, and 2D positioning with lateral offsets for transects, tracklines and trial pits.
- **✓ Future Proofing**: Full support for other linear features, such as tunnels or pipelines, that might be introduced in future versions of DIGGS.

### Core Concepts: The Building Blocks

DIGGS 3.0 linear referencing is built on three fundamental components defined by ISO 19148:

1. **Linear Element (diggs:LinearExtent object, extended in 3.0)**: The geometric feature along which measurements are made—the physical path of the borehole, trackline, or transect—with new properties that define anchor points for controlling interpolation.

2. **Linear Referencing Method (diggs:LinearReferencingMethod object, new in 3.0)**: The procedure or algorithm for determining positions—how measurements are calculated and what units are used.

3. **Linear Spatial Reference System (diggs:LinearSpatialReferenceSystem, extended in 3.0)**: The complete coordinate system created by combining a Linear Element with a Linear Referencing Method. Extended to provide for referents and the new diggs:LinearReferencingMethod.

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
        <datumReference>ground surface</datumReference>
    </LinearExtent>
</centerLine>
```

The gml:posList element holds two 3D coordinate tuples for the path, in the spatial coordinate reference system defined by the srsName attribute. These coordinates describe a vertical borehole at a specific latitude/longitude, extending from elevation 818.6 feet (ground surface) down to 777.6 feet (41 feet total depth). The `datumReference` value derives from an enumerated list and indicates that the origin of the path is at the ground surface.

More complex centerlines that describe an inclined or deviated borehole can be defined by simply altering and/or adding more coordinate tuples to define the path in 3D space. No other properties are necessary for defining complex paths.

![Figure 2: LinearExtent Geometry](figure2_linearextent_geometry.svg)
*Figure 2: LinearExtent defines the geometric path of the borehole with ordered coordinates from top to bottom*

## Linear Referencing Methods

The **LinearReferencingMethod** object defines *how* positions are determined along the linear element. DIGGS 3.0 supports three types per ISO 19148:

| Method Type | Description | Example Use |
|-------------|-------------|-------------|
| **Absolute** | Distance from the linear element origin (the first coordinate) | Measured depth from top of hole |
| **Relative** | Distance from a specified referent point | Depth below ground surface where, for example, the linear element origin is referenced to the drill rig's rotary table |
| **Interpolative** | Position interpolated between control points | Observations on core samples where the cores are stretched or compressed relative to the cored interval |

For the above example, the LinearReferencingMethod object for measured depth down the borehole in feet would be encoded as follows:

```xml
<LinearReferencingMethod gml:id="md_ft">
    <gml:description>
        Absolute measured depth from top of borehole or sounding in feet. Measurements are cumulative 3D distance along the borehole/sounding centerline from the first coordinate (typically top of hole or rig datum) proceeding downward through the hole. 
    </gml:description>
    <gml:identifier codeSpace="https://diggsml.org/def/authorities.xml#DIGGS">md_ft</gml:identifier>
    <name>trueFootage</name>
    <type>absolute</type>
    <units>ft</units>
    <positiveDirection>downward</positiveDirection>
</LinearReferencingMethod>
```


### The LinearReferencingMethod Dictionary: A Time-Saver

Rather than defining linear referencing methods in every data file, DIGGS 3.0 provides a **standard dictionary** of common methods at:

**https://diggsml.org/def/crs/DIGGS/0.1/lrm.xml**

This dictionary includes 16 pre-defined methods covering:

- Borehole and sounding measured depth (absolute and relative, feet and meters)
- Trackline/transect distances (absolute and relative, feet and meters)
- Core sample positioning (top-down, bottom-up, interpolative)
- Trial pit positioning with lateral offsets

Users can reference these by URL in their `LinearSpatialReferenceSystem` definitions, eliminating the need to encode the same `LinearReferencingMethod` objects repeatedly. This promotes consistency and reduces file size.

## Creating a Linear Spatial Reference System

A **LinearSpatialReferenceSystem** combines a `LinearExtent` with an `LinearReferencingMethod` to create a complete coordinate reference system. This is the key object that gets referenced throughout your DIGGS data using `srsName` attributes.

### Example 1: Absolute Measured Depth

Here's how to define a linear reference system for measuring absolute depth from top of hole:

```xml
<linearReferencing>
    <LinearSpatialReferenceSystem gml:id="lsr-B-001-0-20">
        <gml:identifier codeSpace="OGC">lsr-B-001-0-20</gml:identifier>
        <gml:name>Depth reference system for B-001-0-20</gml:name>
        
        <!-- Reference to the borehole's centerline geometry -->
        <glr:linearElement xlink:href="#cl-B-001-0-20"/>
        
        <!-- Reference the LinearReferencingMethod from the standard dictionary -->
        <lrm xlink:href="https://diggsml.org/def/crs/DIGGS/0.1/lrm.xml#md_ft"/>
    </LinearSpatialReferenceSystem>
</linearReferencing>
```

Now, anywhere in your DIGGS file, you can reference this coordinate system:

```xml
<!-- Stiff, grey clay layer from 12 to 18 ft -->
<LinearExtent srsDimension="1" srsName="#lsr-B-001-0-20">
    <gml:posList>12.00 18.00</gml:posList>
</LinearExtent>

<!-- Water level at 9 feet depth -->
<PointLocation srsDimension="1" srsName="#lsr-B-001-0-20">
    <gml:pos>9.00</gml:pos>
</PointLocation>
```

![Figure 3: Absolute Linear Referencing](figure3_absolute_referencing.svg)
*Figure 3: Absolute linear referencing measures all positions from a fixed origin (top of hole)*

### Example 2: Relative Referencing with Referents

Sometimes you need to measure from a point other than the origin. For example, after installing casing, you might want to reference water levels from the top of casing rather than from the original ground surface. This is where **referents** come in.

A **referent** is a reference point along the linear feature. Common referents include ground surface, top of casing, seafloor, kelly bushing, or any other significant datum point. When using a *relative* `LinearReferencingMethod`, you specify which referent serves as the origin (zero point) for measurements.

```xml
<linearReferencing>
    <LinearSpatialReferenceSystem gml:id="lsr-B-001-0-20_casingTop">
        <gml:identifier codeSpace="OGC">lsr-B-001-0-20_casingTop</gml:identifier>
        <gml:name>Depth reference system from top of casing in B-001-0-20</gml:name>
        <glr:linearElement xlink:href="#cl-B-001-0-20"/>
        <lrm>
            <LinearReferencingMethod gml:id="reference_md_ft">
                <gml:description>
                     Measured depth relative to a specified referent location within a borehole or sounding in feet. 
                    The referent (e.g., ground surface, casing top, seafloor) must be defined in the LinearSRS. 
                    Positive direction is downward from the referent.
                </gml:description>
                <gml:identifier codeSpace="https://diggsml.org/def/authorities.xml#DIGGS">reference_md_ft</gml:identifier>
                <name>referenceFootage</name>
                <type>relative</type>
                <units>ft</units>
                <positiveDirection>downward</positiveDirection>
            </LinearReferencingMethod>
         </lrm>
         <referent>
            <Referent gml:id="Referent_top_of_casing">
                <name>TOC</name>
                <referentType>top of casing</referentType>
                <position>
                    <PointLocation gml:id="topOfCasing" srsDimension="3" srsName="https://www.opengis.net/def/crs-compound?1=http://www.opengis.net/def/crs/EPSG/0/4326%262=http://www.opengis.net/def/crs/EPSG/0/5702">
                        <gml:pos>39.474660 -81.796858 816.6</gml:pos>
                    </PointLocation>
                </position>
                <distanceAlong uom="ft">2</distanceAlong>
            </Referent>
        </referent>
    </LinearSpatialReferenceSystem>
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

In the provided example, Borehole B-001-0-20 has **two LinearSpatialReferenceSystem definitions**:

1. **lsr-B-001-0-20**: Absolute measured depth from top of hole (used for drilling operations, backfill intervals, construction methods)
2. **lsr-B-001-0-20_casingTop**: Relative depth from top of casing (used for post-installation water level monitoring)

The initial water strike at 9 feet depth (during drilling) references the absolute system. A follow-up reading one day later, after casing installation, references the relative system at 7 feet below top of casing. Both describe the same physical water level, but use different coordinate systems appropriate to when and how the measurement was made.

> **IMPORTANT:** Having multiple LinearSpatialReferenceSystem definitions for a single borehole is perfectly valid and often necessary. Each represents a different way of measuring position along the same physical feature, appropriate for different phases of work or different measurement conventions.

## Advanced Features for DIGGS 3.0 Linear Referencing

### Anchor Points for Core Stretch/Compression

When core samples are recovered, the physical core length may differ from the cored interval due to expansion (common in clay-rich soils) or compression (from sampling disturbance or desiccation). **Anchor points** provide calibration points to adjust positions between the physical core and the borehole depth system.

For example, if you core from 10-15 feet (5 feet drilled interval) but recover 5.5 feet of core (10% expansion), you can use an *interpolative* `LinearReferencingMethod` with anchor points at the top and bottom of the core. This scales positions proportionally so that observations on the physical core can be accurately tied back to depth in the borehole. Anchor points are defined in the LinearExtent object defined for the core sample.

### 2D Linear Referencing with Lateral Offsets

For features like transects, trial pits, or geophysical tracklines, you may need to record positions that are offset from the centerline. DIGGS 3.0 supports **2D linear referencing** where coordinates include both distance along and lateral offset.

A `LinearReferencingMethod` that includes the `OffsetParameters` object becomes a 2D method. Positions are then encoded as `<gml:pos>distance offset</gml:pos>`, where the first value is distance along the centerline `LinearExtent` and the second is the offset (by default positive to the right when facing the positive centerline direction). The `offsetType` property of the `OffsetParameters` object defines how the offset is measured from the centerline (perpendicular, constantBearing, or radial).

## Migration from DIGGS 2.6

If you're currently using DIGGS 2.6 with the GML 3.3/lr-based linear referencing, here's what you need to know about migrating your instances to DIGGS 3.0:

### Key Changes

- `diggs:LinearExtent`: extended to include `anchorPoint` properties to support interpolative linear referencing methods.
- `diggs:LinearReferencingMethod`: a new object that replaces `glr:LinearReferencingMethod`. It includes geotechnical-specific enumerations for the `name` property and adds properties to support lateral offsets.
- `diggs:LinearSpatialReferenceSystem`: now includes a choice compositor to allow for either the legacy `glr:lrm` property (from GML 3.3/lr that references or holds the `glr:LinearReferencingMethod` object) or a new `diggs:lrm` property (that holds or references the new `diggs:LinearReferencingMethod` object). It also adds `referent` properties to support relative linear referencing methods.

Linear referencing encodings from DIGGS v. 2.6 will validate under 3.0, but the GML 3.3/lr namespace elements `glr:lrm` and `glr:LinearReferencingMethod` are deprecated in v. 3.0. The new `diggs:` namespace elements `diggs:lrm` and `diggs:LinearReferencingMethod` should be used instead. In addition, the Linear Reference Method `name` property value of "`chainage`" has been deprecated and should be replaced with "`trueFootage`" or "`trueMeters`" depending on the `units` value. 

To convert existing 2.6 linear referencing code to 3.0, you only need to replace the `<glr:lrm>` element with the new DIGGS namespace `<lrm>` element. For cases where your borehole positions are absolute measured depths (from top of hole) in feet (or neters), you can take advantage of the standard linear referencing dictionary, which will reduce the encoding to a single line:
```xml
 <lrm xlink:href="https://diggsml.org/def/crs/DIGGS/0.1/lrm.xml#md_ft"/> (#md_m) if depths are in meters
 ```

Processing applications should be updated to take advantage of the new features/enumerations available in version 3.0. Xpath queries that use the local-name() function will correctly retrieve values from both `diggs:` and `glr:` namespace elements.

## Benefits of the New Approach

**Standards Compliance**: Full alignment with ISO 19148:2021, ensuring interoperability with other geospatial systems and future-proofing your data.

**Clarity & Simplicity**: The schema directly reflects geotechnical workflows with clearly defined enumerated values for defining linear reference system properties.

**Enhanced Features**: Support referents, stretched/compressed core samples, and lateral offset positions.

**Reusability**: Standard LinearReferencingMethod dictionary eliminates redundant encoding. Reference common methods by URL instead of duplicating definitions.

## Getting Started

Ready to implement DIGGS 3.0 linear referencing in your projects? Here are your next steps:

1. **Review the Schema**: Until 3.0 is officially released, the linear referencing schema is available on GitHub at https://github.com/DIGGSml/schema-dev/tree/3.0.
2. **Explore the Dictionary**: Browse the standard [LinearReferencingMethod dictionary](https://diggsml.org/def/crs/DIGGS/0.1/lrm.xml)
3. **Review Examples**: Review [this example instance](https://github.com/DIGGSml/schema-dev/blob/3.0/Instances/DocExample3.0.xml) in the DIGGS repository to see linear referencing in action
4. **Join the DIGGS Effort**
    - **Github**: Contribute to our repositories at https://github.com/DIGGSml
    - **Monthly Meetings**: Join our monthly online discussions where we tackle issues like this one. Contact Allen Cadden (acadden@schnabel-eng.com) or Ross Cutts (rcutts@schnabel-eng.com) to receive meeting invites.
    - **Feedback**: Share your experiences implementing these changes

## Conclusion

Linear referencing is fundamental to how we work with geotechnical data. The DIGGS 3.0 update provides a robust, standards-compliant framework that directly serves the needs of our community and will grow with it. By adopting ISO 19148:2021 principles and focusing on geotechnical workflows, we've created a system that's both powerful and intuitive.

Whether you're encoding borehole logs, CPT soundings, geophysical surveys, or any other linear sampling feature, DIGGS 3.0 gives you the tools to precisely, consistently, and interoperably describe where your data comes from. And with the standard LinearReferencingMethod dictionary, implementing linear referencing in your DIGGS files is easier than ever.

We encourage you to explore the new linear referencing schema, experiment with examples, and share your feedback. Together, we're building the future of geotechnical data exchange.
