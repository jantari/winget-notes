# Index V2 Notes

Download: https://cdn.winget.microsoft.com/cache/source2.msix

SQLite DB in the same place, but with differenct content (different tables).
e.g.:
  SELECT *, quote(hash) FROM packages WHERE id = "Microsoft.PowerShell";
  SELECT * FROM norm_publishers2 WHERE package = 2928;
  SELECT * FROM norm_names2 WHERE package = 2928;

With the new DB structure we should now be able to search for packages by tags and names.

No versions in the SQLite DB anymore.
Instead, separate download of MSZIP-compressed Versions-YAML file,
e.g.:
  	https://cdn.winget.microsoft.com/cache/packages/Microsoft.VisualStudioCode/746857b3/versionData.mszyml

URL follows the schema:
  "packages/" + packageIdentifier + "/" + manifestHash.Substring(0, 8) + "/versionData.mszyml"
See:
  https://github.com/JohnMcPMS/winget-cli/blob/43425fe97d237e03026fca4530dbc422ab445595/src/AppInstallerCommonCore/PackageVersionDataManifest.cpp#L75
  
Downloaded file is MSZIP-compressed YAML + 28-byte header in front.
The header is either custom or part of MSZIP or Win Compression API (https://learn.microsoft.com/en-us/windows/win32/api/compressapi/nf-compressapi-createdecompressor) but I could not find definitive info.

Header Example:

0a 51 e5 c0 18 00 ef 02 e8 37 00 00 00 00 00 00
e8 37 00 00 00 00 00 00 93 12 00 00 43 4b <...>

What I found / understood:

0a51
e5c0
1800
ef02
e837 - uncompressed file / block size
0000
0000
0000
e837 - uncompressed file / block size
0000
0000
0000
9312 - compressed data size (file size -28 bytes header, probably just the next block?)
0000
434b - "CK" header

After decompressing we can read the YAML and get a list of available versions with links to the manifests and checksums.


