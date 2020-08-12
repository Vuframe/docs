# Upload Types

The VUFRAME® Public API currently supports the following types of 3D file:

Identifier | Description | File Types
---------- | ----------- | ----------
any_3d | 3D scene exported using the VUFRAME® Unity Plugin | .VF3D, .ZIP
aura3d_files | 3D scene exported using the AuraSDK for Unity | .aura3d
fbx_files | 3D scene in Autodesk FBX format | .FBX, .ZIP containing .FBX and texture files
obj_files | 3D scene in Waveform OBJ format | .ZIP containing .OBJ and .MTL files
c4d_files | 3D scene in Cinema 4D format | .C4D, .ZIP containing .C4D and texture files
vidya_zip | 3D scene exported from Assyst Vidya | .ZIP containing Vidya .OBJ file and textures


<aside class="notice">
Pass one of the identifiers listed above when calling <code>POST /items</code> as the value for the <code>content_type</code> parameter.
</aside>
