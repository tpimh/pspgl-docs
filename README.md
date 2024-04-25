PSPGL
=====

#### (something a bit like) OpenGL for the PSP

Jeremy Fitzhardinge <[jeremy@goop.org](mailto:jeremy@goop.org)\>

Quick Start
-----------

[Download](http://hg.goop.org/pspgl/jsgf?cmd=archive;node=working;type=zip) .zip file of current source.

Check out the source from Subversion with `svn co svn://svn.pspdev.org/psp/trunk/pspgl`

[Pre-compiled demos](pspgl-demos.html) of goop.org PSPGL features.

Contents
--------

*   [Updates](#updates)
*   [Getting the code](#getting-code)
*   [Status](#status)
    *   [improvements](#improvements)
    *   [limitations](#limitations)
    *   [immediate mode](#immediate)
    *   [vertex arrays](#vertex-arrays)
    *   [textures](#textures)
    *   [transform](#transform)
    *   [EGL and GLUT](#egl)
*   [PSP-specific extensions](#extensions)
    *   [vertex pointers](#vertex-pointers)
    *   [GL\_PSP\_statistics](#statistics)
    *   [GL\_PSP\_bezier\_patch](#bezier-patch)
    *   [GL\_PSP\_vertex\_blend](#vertex-blend)
*   [TODO](#todo)
*   [References](#references)

Recent updates
--------------

*   Render to texture implemented, via EGL Pbuffers
*   LINE\_LOOPs now close properly in Begin/End (mrbrown)
*   Fixed glTexSubImage, which was broken.
*   Implemented swizzled textures, which gives some programs a large performance boost.
*   Fixes for switching between two different indexed textures.
*   A big chunk of under-the-hood work to bring render-to-texture a little closer, including VRAM heap compaction.
*   Fixed up initialization, which was surprisingly broken. It should now work reliably on 1.0 PSPs.
*   Implemented glPush/PopAttrib, and glPush/PopClientAttrib. These are not strictly part of EGL, but they're very useful.
*   Added [GL\_PSP\_view\_matrix](#view-matrix) extension to make use of the PSP's view transform matrix. This adds the GL\_VIEW\_PSP matrix, which is selectable with glMatrixMode().
*   Added automatic mipmap generation. This uses the hardware to do mipmap generation, but it only works for RGBA textures (not compressed, luminance/intensity or indexed). Includes an [extension](#mipmap-debug) for mipmap debugging.
*   Implemented gluScaleImage and gluBuild2DMipmaps
*   Fixed problems with mipmaps
*   **SVN PSPGL updated**. I will keep maintaining the Mercurial trees, but the SVN version on pspdev.org is now up to date. Check it out with `svn co svn://svn.pspdev.org/psp/trunk/pspgl`
*   Various API additions to get ODE's demos to compile.
*   Improved monochrome textures to use an internal cmap to reduce the size of the internal form of the textures (they're native now)
*   Implemented [PSP\_vertex\_blend](#vertex-blend) extension
*   Implemented [spline patch](#bezier-patch) surfaces
*   Implemented indexed forms of bezier and spline patches
*   Cleaned up varray code. This doesn't have any visible effect, but it does simplify the vertex array code and makes it easy to add other indexed primitvies (ie, beziers)
*   Implemented glDrawBezierArrays(). Still to do: indexed arrays. Details [below](#bezier-patch)
*   fixed spotlights; documented spot exponent and cutoff angle
*   added magnifier test program for glCopyTexImage2D
*   implement glCopyTexImage2D - pretty good performance
*   fixed bug in native vertex format detection
*   fixed bugs in glInterleavedArrays
*   fixed handling of pinned buffer; removes performance problem with index buffer objects

Introduction
============

PSPGL is an OpenGL-like library for the Sony PSP. It was started by Holger Wächtler, but I ([Jeremy Fitzhardinge](mailto:jeremy@goop.org?subject=PSPGL)) have done a lot of work on it. As of now, the SVN version of PSPGL has all my changes in it.

This document assumes you're familiar with [OpenGL](http://www.opengl.org/). If not, you can find the specs at opengl.org, and many tutorials around the net.

Aims
----

My goal with this library is to provide a efficient, useful and (relatively) complete subset of OpenGL which makes all the PSP's hardware abilities available, either through standard OpenGL mechanisms or with extensions. Where PSPGL and OpenGL are the same, they should behave the same.

My main influence is the OpenGL/ES subset of OpenGL. This provides a fairly complete API, but excludes a lot of the more esoteric corners of OpenGL which don't fit small devices. PSPGL already provides a superset of OpenGL/ES by providing calls such as `glBegin`. In general I'll consider such extensions if they're very widely used or simple to implement, but not for the sake of it (so, `glPushAttrib` is likely simply because its heavily used, but display lists are less likely).

Getting the code
================

The pspdev.org Subversion server is probably the best way to get the code. You can [browse it](http://svn.pspdev.org/listing.php?repname=psp&path=%2Ftrunk%2Fpspgl%2F&rev=0&sc=0) or check it out with Subversion (`svn co svn://svn.pspdev.org/psp/trunk/pspgl`).

I locally maintain the code with the [Mercurial](http://selenic.com/mercurial) source control system. This is a simple, portable system which runs on Unix, Mac and Windows systems. If you want to hack on PSPGL's code, then the best thing to do is install Mercurial, and pull a current tree from [http://hg.goop.org/pspgl/jsgf](http://hg.goop.org/pspgl/jsgf). The tag "working" marks the version I last tested as working reasonably well (ie, not completely broken); it may or may not be the tip. Use `hg update -C working` to get this version.

All the source is browsable at [http://hg.goop.org/pspgl/jsgf](http://hg.goop.org/pspgl/jsgf). You can download the complete source archive of the "tip" version as a [.zip](http://hg.goop.org/pspgl/jsgf?cmd=archive;node=working;type=zip) or [.tar.gz](http://hg.goop.org/pspgl/jsgf?cmd=archive;node=working;type=gz) file.

I also maintain a separate archive of my changes as patches at [http://hg.goop.org/pspgl/patches](http://hg.goop.org/pspgl/patches). Again, these can be downloaded as a [zip](http://hg.goop.org/pspgl/patches?cmd=archive;node=tip;type=zip) or [.tar.gz](http://hg.goop.org/pspgl/patches?cmd=archive;node=tip;type=gz) file. The `series` file lists what order the patches should be applied in.

Status
======

PSPGL is very incomplete. I've tried to make what has been implemented work correctly in a useful way, but there are likely lots of bugs. This is a discussion of what does work (or at least, how its supposed to work). The [TODO](#todo) section below talks about what I'm planning.

Please report any bugs you find to [me](mailto:jeremy@goop.org?subject=PSPGL%20bug), preferably with some sample code which demonstrates the problem. A register state dump is also very useful (enable these by setting the `#if 0` to `#if 1` around line 42 of pspgl\_misc.h and recompiling PSPGL+your program).

Features of PSPGL
-----------------

A quick list of PSPGL features:

texture format conversion

PSPGL accepts a wide range of common texture formats, and will convert them into hardware format.

compressed/paletted textures

vertex array format conversion

You can use the normal `gl*Pointer` calls to configure arrays in any way you like (though using native hardware layout will be most efficient)

compiled vertex arrays

The CVA extension allows vertex array copying/conversion to be cached for multiple uses of an array.

vertex buffer objects

VBOs allow vertex data to be used directly without copying or conversion.

EGL config selection works

You can choose 16 or 32 bit framebuffers in a number of configurations, and decide whether or not to have a depth buffer

[extensions](#extensions) for PSP hardware features

GL extensions allow PSP hardware features to be exposed in an efficient way without breaking programs which don't know/care abou them.

Automatic mipmap generation in hardware

General limitations
-------------------

Reading back state is not supported. Some state is readable, but a lot is not. Use the `glGetIntegerv`/`glGetFloatv` call appropriate to the type of data you want to fetch; they do not return the same set of state.

Complex conversions are not supported. It will perform enough data conversion to let you get away with not knowing the precise hardware format, but it won't do anything heroic. Things like automatic compression are right out.

Display lists are not supported. There is some non-functioning vestigial support. I'm still unsure whether they can be made to work in a sufficiently efficient and lightweight way.

Immediate mode geometry
-----------------------

PSPGL supports specifying geometry in immediate mode within a `glBegin/glEnd` pair. Immediate mode is useful for writing programs quickly, debugging, instrumentation, etc, but it is never going to be a high-performance path.

The following primitives are supported:

`GL_POINTS`

Only 1 pixel points

`GL_LINES`

Only 1 pixel lines

`GL_LINE_STRIP`

OK

`GL_LINE_LOOP`

Behaves the same as a LINE\_STRIP: the closing edge isn't drawn

`GL_TRIANGLES`

OK

`GL_TRIANGLE_STRIP`

OK

`GL_TRIANGLE_FAN`

OK

`GL_QUADS`

not supported

`GL_QUAD_STRIP`

not supported

`GL_POLYGON`

not supported

Vertex arrays
-------------

Vertex arrays are the preferred interface for submitting geometry information. Directly using OpenGL 1.1 vertex arrays will have a performance advantage over immediate mode simply because there are fewer function calls, but there are higher performance techniques.

`glArrayElement` offers little advantage over the normal immediate mode calls; use `glDrawArrays`, or `glDrawElements`.

Use the smallest vertex types possible: use GL\_UNSIGNED\_BYTE rather than FLOAT for colours; use SHORTs rather than FLOATs for vertex and texcoords. Don't enable unused arrays (for example, don't enable a normal array unless lighting is enabled; there are exceptions though). Use BYTE or SHORT indices rather than INT. Use the [native vertex format](#native-vertex) where possible.

Use strips and fans where possible, though indexed independent triangles are still pretty efficient.

Note: there's no need to do any explicit cache flushes or use uncached pointers when passing vertex pointers into PSPGL. PSPGL will do all the appropriate cache management for itself.

### EXT\_compiled\_vertex\_arrays

PSPGL supports the CVA extension. This allows you to set up the vertex arrays, and then call `glLockArraysEXT`. This will cause PSPGL to convert the vertex data into hardware form, and cache it in the PSP GE's local memory. You may then use the vertex data with multiple calls to `glDrawArrays`/`glDraw(Range)Elements`.

There's probably no advantage in using this unless you use the array multiple times, and don't lock too many unused vertices (ie, only lock the part of the arrays you're actually using). You must unlock the arrays with `glUnlockArraysEXT` before updating the array pointers with `gl*Pointer`; if you don't, the pointer update will be effectively ignored. Similarly, changing the contents of a locked array will have no effect.

It will only lock up to 128k of vertex data. If you try to lock more, the call will have no effect. Try to keep your vertex arrays under this limit, or use a vertex buffer object in native form.

### ARB\_vertex\_buffer\_objects

(_Note:_ you'll need to carefully read the specification of the ARB\_vertex\_buffer\_object extension \[see [references](#references)\] to understand the discussion below. You can safely ignore all this though.)

PSPGL implements the VBO extension (now part of OpenGL 1.5). Buffer objects are a powerful general way for an application to have more control over how OpenGL allocates and uses memory, and are used for more then just vertex arrays. However, vertex buffer objects are very useful.

Buffer objects are primiarily useful when you arrange your data into a form which is directly usable by the hardware without any conversion. If you do this, then a buffer object allows the hardware to directly DMA the data out of the buffer without conversion or copying. Unfortunately the PSP is a bit more rigid than other 3D hardware about what vertex and texture formats and arrangements in memory it will accept, so this takes some care.

PSPGL currently ignores the "usage" parameter of `glBufferDataARB`, but the intention is that it will be used to decide whether data will be placed in system memory or EDRAM (though ultimately buffers will tend to migrate between the two memory pools).

Because the PSP uses a MIPS CPU without coherent caches, the caches must be managed in software. PSPGL will do this for you, but only if you use the API correctly. This means that you have to be careful to use the `glMapBufferARB`/`glUnmapBufferARB` functions properly. PSPGL tries hard to raise GL errors if you use a mapped buffer as an argument to a GL call, so be sure to check for errors when debugging VBO code. **NEVER** use a buffer pointer after you've unmapped it.

The "access" parameter of `glMapBufferARB` is used to determine how the cache is treated for the new memory:

Mapping access

Mapping type

Map action

Unmap action

Notes

`GL_WRITE_ONLY_ARB`

Uncached

sync with hardware if busy

\-

The mapping is uncached to help prevent cache pollution; reads will work from a write-only mapping, but they'll probably be very slow. If you're replacing the entire contents of the buffer while the hardware is potentially using it, it is more efficient to use `glBufferDataARB` to replace the buffer with a new one, because this doesn't require waiting on the hardware.

`GL_READ_ONLY_ARB`

Cached

\-

cache is invalidated but not flushed

This will still be writable, but writing to it may cause very strange, non-deterministic results; the cache lines may not be flushed to memory for the hardware to see, or they may be discarded without ever being written (giving the appearance of data which "sticks" for a while, but then reverts to its old value).

`GL_READ_WRITE_ARB`

Cached

sync with hardware if busy

dirty lines are flushed, and the cache is invalidated

Safe for all usage, but not as efficient. Use only if you really need to have read-write access to the memory.

In general, the assumption is that buffer objects are intended for buffers shared with hardware. They are kept in CPU cache only while mapped for access by the CPU, and are otherwise evicted from the cache. This leaves the CPU cache free for other data, and makes sure the hardware always sees a consistent view of the memory. If you don't put your arrays in buffer objects into a form which is directly useful to hardware, you end up using buffer objects like an inefficient form of `malloc` with bad cache characteristics.

Note that it is always an error to map a buffer while it is still in use; PSPGL enforces this by raising an error if you try to use a buffer while its mapped, and waiting for the hardware to finish if you create a writable mapping.

To arrange a vertex array in native vertex format, you must specify your arrays in the following order in memory (leaving out any array you're not enabling), with sizes and types as follows:

Array

Types

Size

GL\_TEX\_COORD\_ARRAY

GL\_BYTE, GL\_SHORT, GL\_FLOAT

2

GL\_WEIGHT\_ARRAY\_PSP

GL\_BYTE, GL\_SHORT, GL\_FLOAT

1-8

GL\_COLOR\_ARRAY

GL\_UNSIGNED\_BYTE

4

GL\_NORMAL\_ARRAY

GL\_BYTE, GL\_SHORT, GL\_FLOAT

3

GL\_VERTEX\_ARRAY

GL\_BYTE, GL\_SHORT, GL\_FLOAT

3

See [the Wiki](http://wiki.ps2dev.org/psp:ge_vertex_format) for more details.

_Note:_ Your array of vertices must be packed, so there is no non-vertex data between each vertex. Also, you must enable all these arrays with `glEnableClientState` for them to be considered as part of the "native format" check. For example, if you sometimes need normals and sometimes not, then it is probably better to always keep the normal array enabled; if you disable them when you disable lighting, then PSPGL needs to copy and re-pack your arrays when you use them, which is likely more expensive than simply letting the hardware ignore an unused normal element in each vertex.

You may also put index data into a buffer object, using the `GL_ELEMENT_ARRAY_BUFFER_ARB` target. This is pretty straightforward. The only constraint is that you use `GL_UNSIGNED_BYTE` or `GL_UNSIGNED_SHORT` as your index type; the hardware doesn't seem to support 32-bit indices.

Textures
--------

PSPGL supports only 2D textures with power-of-two dimensions. 1D textures can be easily emulated with a Nx1 (or 1xN) texture; 3D textures are just not available.

PSPGL will convert textures you provide into the native hardware format, but it will only perform a limited range of conversions. In general, it will rearrange bits in a pixel format, but it won't convert the type or format of a texture. Therefore, if you want an `GL_RGB` internal format, you must provide a `GL_RGB` format texture.

`glTexImage2D` and `glTexSubImage2D` work mostly as expected, though `glPixelStore` has not been implemented, so all textures are assumed to be tightly packed in contigious memory (no 4-byte rounding for each row either). PSPGL will make a copy of your texture data, so you can free/overwrite/reuse your copy immediately. PSPGL will also manage cache flushing, etc, so you don't need to.

Mipmapping mostly works as expected. GL\_GENERATE\_MIPMAPS texture parameter is supported, so you can get the hardware to generate mipmaps for you. It also supports a GL\_GENERATE\_MIPMAP\_DEBUG\_PSP flag, which will add tinting to each mipmap level to make them easier to see. Automatic mipmap generation only works for RGBA texures - not compressed, indexed or luminance/intensity formats.

The PSP hardware seems to have a bug where a texture viewed from a particular angle will use larger mipmaps than necessary, which means the appearance will change on screen as view angle changes, and there may be a performance impact.

The following types and formats are supported:

Format

Type

Hardware bytes/pixel

GL\_RGB

GL\_UNSIGNED\_BYTE

4

GL\_RGB

GL\_UNSIGNED\_SHORT\_5\_5\_5\_1

2

GL\_RGB

GL\_UNSIGNED\_SHORT\_5\_6\_5

2

GL\_RGB

GL\_UNSIGNED\_SHORT\_4\_4\_4\_4

2

GL\_RGB

GL\_UNSIGNED\_SHORT\_1\_5\_5\_5\_REV

2\*

GL\_RGB

GL\_UNSIGNED\_SHORT\_5\_6\_5\_REV

2\*

GL\_RGB

GL\_UNSIGNED\_SHORT\_4\_4\_4\_4\_REV

2\*

GL\_BGR

GL\_UNSIGNED\_SHORT\_5\_6\_5

2\*

GL\_RGBA

GL\_UNSIGNED\_BYTE

4\*

GL\_RGBA

GL\_UNSIGNED\_SHORT\_5\_5\_5\_1

2

GL\_RGBA

GL\_UNSIGNED\_SHORT\_4\_4\_4\_4

2

GL\_RGBA

GL\_UNSIGNED\_SHORT\_1\_5\_5\_5\_REV

2\*

GL\_RGBA

GL\_UNSIGNED\_SHORT\_4\_4\_4\_4\_REV

2\*

GL\_ABGR\_EXT

GL\_UNSIGNED\_SHORT\_4\_4\_4\_4

2\*

GL\_LUMINANCE\_ALPHA

GL\_UNSIGNED\_BYTE

4

GL\_LUMINANCE

GL\_UNSIGNED\_BYTE

1\*

GL\_ALPHA

GL\_UNSIGNED\_BYTE

1\*

GL\_INTENSITY

GL\_UNSIGNED\_BYTE

1\*

Formats marked with \* are native and require no conversion.

EXT\_paletted\_texture
----------------------

The PSP hardware supports paletted textures, and PSPGL exposes that with the EXT\_paletted\_texture extension. Each texture object has its own colour map, which is set with `glColorTableEXT` using the `GL_TEXTURE_2D` target. The colour table can be in any of the `GL_RGB` or `GL_RGBA` format/type combinations supported for textures. The paletted textures themselves can use a format of `GL_COLOR_INDEX4_EXT`, `GL_COLOR_INDEX8_EXT` or `GL_COLOR_INDEX16_EXT`. There's no way to share a colour table between textures.

`glTexSubImage2D` does not work properly on 4-bit/texel paletted textures.

GL\_ARB\_texture\_compression and GL\_EXT\_texture\_compression\_dxt1
---------------------------------------------------------------------

The PSP hardware supports DXTn (aka S3TC) compressed textures. PSPGL implements the `glCompressedTexImage2D` call to allow compressed textures to be copied into GL. It supports these compressed formats:

*   GL\_COMPRESSED\_RGB\_S3TC\_DXT1\_EXT
*   GL\_COMPRESSED\_RGBA\_S3TC\_DXT1\_EXT
*   GL\_COMPRESSED\_RGBA\_S3TC\_DXT3\_EXT
*   GL\_COMPRESSED\_RGBA\_S3TC\_DXT5\_EXT

`glCompressedTexSubImage2D` has not been implemented.

Transform, etc
--------------

Texture matrix transforms don't work.

Texture coord generation (used for environment mapping, projectors, etc) are not implemented. It seems from looking at the implementation of environment mapping with libgu, some of the lights are used as parameters for texcoord generation, which means that it will interact with lighting (you lose some lights while generating texture coords).

EGL and GLUT
------------

When you create a GL context with EGL, you can specify what configuration of framebuffers you want bound to that context. You can find an appropriate configuration with `eglChooseConfig`, which will return a number of configurations which match the attributes you specify. If there are no valid configurations, then it will return none.

You can use this mechanism to specify whether you need an alpha channel, how many bits of RGB channel, whether you need a depth buffer or a stencil buffer. Note that in the PSP hardware, the stencil buffer and the destination alpha channel share the same space, so you can have one or the other for a given GL context.

GLUT is simpler to use, and so does not let you specify fine details of the GL configuration; it will always request 8 bits of RGB. You can use the GLUT\_RGB, GLUT\_ALPHA, GLUT\_STENCIL and GLUT\_DEPTH flags to `glutInitDisplayMode` to specify a buffer configuration.

PSP-specific extensions
=======================

PSPGL implements a number of extensions to OpenGL. Some of these subtle extensions of existing behaviour, and some are full OpenGL extensions.

gl\*Pointer
-----------

The `gl*Pointer` functions take a different set of types, which (mostly) match the hardware's capabilities:

Function

Types

`glVertexPointer`

GL\_BYTE, GL\_SHORT, GL\_FLOAT

`glTexCoordPointer`

GL\_BYTE, GL\_SHORT, GL\_FLOAT

`glColorPointer`

GL\_UNSIGNED\_BYTE, GL\_FLOAT (not-native)

`glNormalPointer`

GL\_BYTE, GL\_SHORT, GL\_FLOAT

A future extension will be to allow `glColorPointer` to take a variety of packed colour formats, to make better use of the PSP's vertex colour formats.

GL\_PSP\_statistics
-------------------

This is an extension which is return in the GL\_EXTENSIONS string. It allows an application to get various performance meaurements out of PSPGL. It defines the following functions:

`glEnableStatsPSP(GLenum)`

This enables one aspect of statistics gathering. Currently, the only value it accepts is `GL_STATS_TIMING_PSP`.

`glDisableStatsPSP(GLenum)`

Disable statistics gathering.

`glResetStatsPSP(GLenum)`

Reset a particular counter or timer. Accepts the values:

GL\_STATS\_CMDISSUES\_PSP

Reset the "command issue" counter.

GL\_STATS\_QUEUEWAITTIME\_PSP

Reset the "queue wait time" timer.

`glGetStatisticsuivPSP(GLenum, GLuint *)`

Return a measturement. Values available are:

GL\_STATS\_FRAMETIME\_PSP

Return the total frame time from the end of swap-buffers to swap-buffers for the previous frame.

GL\_STATS\_APPTIME\_PSP

Return the amount of time between the end of one swap-buffers to the start of the next.

GL\_STATS\_SWAPTIME\_PSP

Return the amount of time spent in the last swap-buffers.

GL\_STATS\_CMDISSUES\_PSP

Return the number of buffer flushes since it was last reset.

GL\_STATS\_QUEUEWAITTIME\_PSP

Return the amount of time spent waiting for the graphics processor to finish a queue of commands.

All times are in microseconds.

GL\_PSP\_bezier\_patch
----------------------

This extension exposes the PSP's hardware bezier patch drawing abilities. Bezier patches may only be drawn using vertex arrays; there is no immediate mode interface.

Bezier patches are composed of 4x4 control points. You may specify control point arrays larger than 4x4, but they are used in groups of 4x4, with adjacent patches sharing a set of control points. The size of the array should be of the form 4+3n; the hardware will ignore any left-over control points.

The primitives emitted by the hardware when subdividing the patch are treated like primitives specified normally; they are textured, lit, culled. depth tested, etc as usual. Lighting is computed at the subdivided vertices, and so using a patch is an efficient way to tesselate a surface for high-quality lighting.

Functions:

`void glDrawBezierArraysPSP(Glenum mode, GLuint u, GLuint v, GLint first)`

This function draws the patch, using a u by v array of control points. `mode` specifies which primitive is to be used for the patch; it may be one of GL\_TRIANGLES, GL\_LINES or GL\_POINTS.

`void glDrawBezierElementsPSP(GLenum mode, GLuint u, GLuint v, GLenum idx_type, const GLvoid *indices)`

`void glDrawBezierRangeElementsPSP(GLenum mode, GLuint u, GLuint v, GLenum idx_type, const GLvoid *indices)`

The obvious indexed version of glDrawBezierArrays

`void glPatchSubdivisionPSP(GLuint u, GLuint v)`

Sets the level of subdivision performed for each patch. The default value for u and v is 4. 64 seems to be the upper limit.

`void glDrawSplineArraysPSP(GLenum mode, GLuint u, GLuint v, GLenum uflags, GLenum vflags, GLint first);`

This draws a spline patch. This is much like a bezier patch, but the whole patch is smooth, rather than being smooth in 4x4 sub-patches. The two flags specify whether the patch is intended to abut another spline patch. There are four flag combinations: GL\_PATCH\_INNER\_INNER\_PSP, GL\_PATCH\_INNER\_OUTER\_PSP GL\_PATCH\_OUTER\_INNER\_PSP and GL\_PATCH\_OUTER\_OUTER\_PSP. Each of these specified whether the (u,v)=(1,0) is an inner edge or an outer edge. An inner edge is only drawn up to the 2nd last row of control points; an abutting patch needs to have 3 rows of control points to match up properly. An outer edge goes out to the edge of the control mesh.

`void glDrawSplineElementsPSP(GLenum mode, GLuint u, GLuint v, GLenum uflags, GLenum vflags, GLenum idx_type, const GLvoid *indices);`

`void glDrawSplineRangeElementsPSP(GLenum mode, GLuint start, GLuint end, GLuint u, GLuint v, GLenum uflags, GLenum vflags, GLenum idx_type, const GLvoid *indices);`

The indexed form of glDrawSplineArraysPSP

TODO:

*   special patch culling modes?

GL\_PSP\_vertex\_blend
----------------------

This extension exposes the PSP's vertex blending (or "skinning") capabilies. When this mode is enabled with `glEnable(GL_VERTEX_BLEND_PSP)`, and a weight array is specified and enabled with `glWeightPointerPSP` and `glEnableClientState(GL_WEIGHT_ARRAY_PSP)`, each vertex is transformed by the matrices `GL_BONE[0-7]_PSP`, weighted by the per-vertex weights.

The `GL_BONE[0-7]_PSP` matrices are normal matricies; they are selected with `glMatrixMode`, transformed with the normal functions, etc. The only exception is that `glPushMatrix` will always fail, because they have a max stack depth of 1.

The MODELVIEW matrix still works as expected; it is applied to the result of the weighted summed transforms of the BONE matrices.

Vertex blending is not supported for immediate mode (`glBegin/End`). Code which mixes immediate mode and vertex arrays while using vertex blending will get unexpected results.

Functions:

`void glWeightPointerPSP(GLint size, GLenum type, GLsizei stride, const GLvoid *array)`

Sets the weight pointer. `size` may be 1 to 8, and type may be GL\_BYTE, GL\_SHORT or GL\_FLOAT. Weights must go between texture coords and colour to get a [native vertex](#native-vertex) format.

GL\_PSP\_view\_matrix
---------------------

The PSP transform pipeline contains an extra matrix above OpenGL's transform pipeline. This matrix corresponds to the camera transform, and is applied after the modelview transform. It seems to be mostly redundant, in that you can incorporate it into the modelview matrix without loss of functionality; it might save on some matrix multiplies though. The view matrix also transforms lights, so it allows moving the viewpoint without having to respecify all your lights.

You can operate on this matrix by passing `GL_VIEW_PSP` to `glMatrixMode`.

GL\_PSP\_mipmap\_debug
----------------------

PSPGL supports a debugging mode for mipmap generation, where each mipmap level is tinted to distinguish it from its neighbouring mipmaps. You can enable this mode for a texture by setting the `GL_GENERATE_MIPMAP_DEBUG_PSP` flag with `glTexParameter[if]`. It only has an effect when `GL_GENERATE_MIPMAP` is enabled.

TODO
====

Rough list of things which come to mind. [Tell me](mailto:jeremy@goop.org?subject=PSPGL%20suggestions) of anything else you think of.

Buffer objects
--------------

*   extension to allow an application buffer to be used as the buffer object data storage
*   Automatic placement of buffer data in either system memory or EDRAM (solves texture residency, array memory usage, framebuffers, etc all at once).

Textures
--------

*   test mipmapping more, LOD bias
*   render to texture
*   EGL pbuffers?

Vertex arrays
-------------

*   Support packed colour format arrays.

Transform
---------

*   Implement/test fog
*   Fix texture coord transforms.
*   Implement texgen, env mapping, projective textures, etc
*   Support vertex blending (aka skinning/bones). ARB\_vertex\_blending is almost right, but doesn't really match the PSP's abilities; implementing it is a pain. Probably roll our own. Probably only support for arrays.
*   Support mesh morphing/interpolation. ATI\_vertex\_streams is close enough to use as inspiration. Probably only support for arrays.

Raster primitives
-----------------

*   Expose sprites

References
==========

1.  [OpenGL 1.5 Spec](http://www.opengl.org/documentation/specs/version1.5/glspec15.pdf)
2.  [OpenGL/ES 1.1 spec](http://www.khronos.org/cgi-bin/fetch/fetch.cgi?opengles_spec_1_1)
3.  [EGL 1.2 spec](http://www.khronos.org/cgi-bin/fetch/fetch.cgi?egl_1_2)
4.  [ARB\_vertex\_buffer\_object](http://oss.sgi.com/projects/ogl-sample/registry/ARB/vertex_buffer_object.txt)
5.  [ARB\_texture\_compression](http://oss.sgi.com/projects/ogl-sample/registry/ARB/texture_compression.txt)
6.  [EXT\_texture\_compression\_s3tc](http://oss.sgi.com/projects/ogl-sample/registry/EXT/texture_compression_s3tc.txt)
7.  [EXT\_paletted\_texture](http://oss.sgi.com/projects/ogl-sample/registry/EXT/paletted_texture.txt)
8.  [EXT\_compiled\_vertex\_array](http://oss.sgi.com/projects/ogl-sample/registry/EXT/compiled_vertex_array.txt)
9.  [ARB\_vertex\_blend](http://oss.sgi.com/projects/ogl-sample/registry/ARB/vertex_blend.txt)
10.  [PS2dev.org Wiki](http://wiki.ps2dev.org/doku.php)
