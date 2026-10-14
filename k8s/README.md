# K8s manifests for Geocoding Benchmark - 2024-09-23  

These files are for reference only. It is beyond the scope of this repository to fully recreate the geocoding environment used. Even if the instructions below bring up the kubernetes environment correctly, you will still have to install the data files for nominatim and postgis for the geocoders to work. The relevant doccumentation can be found on the [nominatim install](https://nominatim.org/release-docs/latest/admin/Installation/) page and the [postgis tiger geocoder](https://gitea.osgeo.org/postgis/postgis_tiger_geocoder) github. Degauss is fairly straightforward but for this implementation requires a fastAPI connector hosted behind authentication on dockerhub. If you are keen on learning more reach out.

With that said, if you have docker desktop with kubernetes enableb or another version version of kubernetes running on a local machine (i.e. colima + kind). You can implement this configuration (all commands are run from this directory).

#### edit the \*.pv files  

Make sure {{all_named_directories}} are replaced with the absolute paths on your local file system. On windows prefix the path with: /run/desktop/mnt/host/\<drive\>/.

- local-data-pv.yaml
- local-nominatim-flatnode-pv.yaml
- local-nominatim-postgres-pv.yaml
- local-postgis-geocode-pv.yaml

#### create a namespace

In powershell or the terminal:
```bash
kubectl create namespace gdsc
```

#### apply all mainfests

mac/linux:
```bash
for manifest in $(ls -r ./*.yaml)
do
  kubectl apply -f $manifest
done
```

windows:
```bash
Get-ChildItem -Path ".\" -Filter "*.yaml" |
ForEach-Object {
  kubectl apply -f $_.FullName
}
```

### Contact:

Timothy Norris
University of Miami Libraries
Frost Institute for Data Science and Computing