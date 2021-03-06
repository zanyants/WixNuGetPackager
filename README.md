# WixNuGetPackager

WixNuGetPackager enables the distribution of Windows Installer XML (WiX) libraries and extensions using NuGet packages. WixNuGetPackager is itself distributed as a NuGet package which can be installed in WiX library or extension projects. Once installed in a project, WixNuGetPackager extends the build process to generate a NuGet pacakge containing the compiled artefacts. This generated NuGet package can then be distributed and added to other WiX projects.

## Using WixNuGetPackager

### Creating a NuGet package for distributing a WiX library or extension:

1. Install the `WixNuGetPackager` package from [nuget.org](https://www.nuget.org/packages/WixNuGetPackager/) to your WiX library or extension project. With older versions of Visual Studio, you may need to close and reopen the containing solution for the build process extensions to take effect.
2. In Visual Studio, in the Solution Explorer right click on your WiX library project.
3. Select `Add...` -> `Existing item`
4. Navigate to the NuGet pacakges directory for your solution, typically `[solution_directory]\packages`.
5. Navigate to `WixNuGetPackager.X.Y.Z\tools` (where X.Y.Z is the installed version of the `WixNuGetPackager` package).
6. Ensure that the file type filter dropdown is set to `All files (*.*)`.
7. Select the file `NuGetPackageMetadata.nuspec.template` and click `Add`.
7. Rename the added `NuGetPackageMetadata.nuspec.template` item to `NuGetPackageMetadata.nuspec`
8. Modify the content of the `NuGetPackageMetadata.nuspec` file in your project to provide the metadata for your wixlib NuGet package.
9. Build your project. By default, a copy of `nuget.exe` will be downloaded on the first build, and the generated NuGet pacakge will be copied to the project's output directory (for example, `bin\debug`). These behaviours can be changed, see *Configuration* below.

### Extension packages

Certain conventions and limitations are assumed when packaging an extension project:

* Referenced Wix libraries are ignored. They are assumed to be merged into the project's output assembly (typically as an embedded resource).
* CustomAction DLLs are assumed to be linked into a Wix library, which will be ignored (see above).
* Dependencies on arbitary .NET assemblies (eg, from outside the .NET framework) are currently not supported (it's up to you somehow merge them in to the project's output assembly, eg. ILMerge, EmbeddedResource + AssemblyResolve handler)
* References to other NuGet packages are currently not supported, and will not be expressed as dependencies within the generated nuget package.
* XML schemas are expected to be `EmbeddedResource` items with a `.xsd` extension. All such items will be added to the generated package and exposed to any project consuming the generated package. Schema-aware Intellisense should work. 

### Using a WiX library or extension NuGet package

To use a NuGet package created by WixNuGetPackager from a WiX library or extension project (as above), just add the package to any WiX project that supports `WixLibrary` or `WixExtension` references as appropriate. This has been tested with WiX Setup projects, but should also work with WiX Library and WiX Merge Module projects. The packages WixNuGetPackager creates also extend the build process, so with older versions of Visual Studio you may need to close and reopen the containing solution for the build process extensions to take effect.

## Configuration

The behaviour of WixNuGetPackager can be controlled using various [MSBuild properties](https://msdn.microsoft.com/en-us/library/ms171458.aspx). This section assumes that you are familiar with the basics of MSBuild.

### Configuring the build-time behaviour of the WixNuGetPackager package

`$(WixNuGetPackageId)`
> The identity of the generated NuGet package. Also passed to `nuget pack` as the `$id$` nuspec replacement token.<br/>
> Defaults to `$(OutputName)` (defaulted if unset within the `WixNuGetPackagerPrepare` target)

`$(WixNuGetPackageVersion)`
> If not empty, passed to `nuget pack` as the `$version$` nuspec replacement token.

`$(WixNuGetPackageAuthor)`
> If not empty, passed to `nuget pack` as the `$author$` nuspec replacement token.

`$(WixNuGetPackageDescription)`
> If not empty, passed to `nuget pack` as the `$description$` nuspec replacement token.

`$(WixNuGetPackagerMetadataNuspecFilePath)`
> The fully-qualified to the `nuspec` format file from which (only) the `<metadata>` element will be taken.<br/>
> Defaults to `%(FullPath)` of a single `@(Content)` item with filename `NuGetPackageMetadata.nuspec`.<br/>
> If the project does not contain exactly one such `@(Content)` item, this property is required.

`$(WixNuGetPackagerEnabled)`
> Should WixNuGetPackager do it's thing?<br/>
> `true` (default) or `false`

`$(WixNuGetPackagerDownloadNuGetExe)`
> Should a copy of `nuget.exe` be downloaded on the first build?<br/>
>`true` or `false`<br/>
> Defaults to `true` if `$(WixNuGetPackagerNuGetExePath)` is undefined; otherwise defaults to `false`

`$(WixNuGetPackagerNuGetExePath)`
> The fully-qualified path to `nuget.exe`<br/>
> Ignored if `$(WixNuGetPackagerDownloadNuGetExe)` is `true`; otherwise required.

`$(WixNuGetPackagerOutputDirectory)`
> The directory into which a copy of the file(s) generated by `nuget pack` should be copied.<br/>
> Defaults to `$(OutDir)`. See also `$(WixNuGetPackagerCopyOutputToOutputDirectory)`.

`$(WixNuGetPackagerCopyOutputToOutputDirectory)`
> Should the file(s) generated by `nuget pack` be copied to `$(WixNuGetPackagerOutputDirectory)`?<br/>
>`true` (default) or `false`

### Configuring the build-time behaviour of packages generated by the WixNuGetPackager package

There is currently no configuration available for the build-time behaviour of packages generated by the `WixNuGetPackager` package.

## Advanced MSBuild interaction

This section is intended for advanced MSBuild users.

Targets `WixNuGetPackagerPrepare` and `WixNuGetPackagerCreatePackage` are intended to be stable in name and function and can be referenced elsewhere.

If executed, target `WixNuGetPackagerCreatePackage`:
* adds `@(WixNuGetPackageIntermediatePackage)` for the files output from `nuget pack` to the intermediate directory with extension `.nupkg`
* adds `@(WixNuGetPackagerOutputFiles)` items files copied to the output directory (unless configured not to copy)
* adds `@(WixNuGetPackagerOutputPackage)` containing `@(WixNuGetPackagerOutputFiles)` with extension `.nupkg`

## License

WixNuGetPackager is licensed under the Apache License, Version 2.0 (the "License"); you may not use WixNuGetPackager except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

## Disclaimer
This project was developed for in-house use at [Zany Ants](http://zanyants.com) with limited and specific use cases, and has been released publically in the hope that others will find it useful too. WixNuGetPackager has been tested with Visual Studio 2015, NuGet 2.8.7 and WiX 3.10.0. Your mileage with other versions may vary. It almost certainly will not work with versions of Visual Studio prior to 2010 (MSBuild 4.0 is required), or WiX versions prior to 3.x. It has been designed defensively for NuGet version 3.x, but this has not been tested.

## To Do

* Provide a PowerShell script equivalent to steps 2-7 in the *Using WixNuGetPackager* section which could be run from the Package Manager Console.
* Support packaging other WiX project types in a meaningful way.
* Test with other versions of Visual Studio, NuGet and WiX.