# Make sure Packer's Masterless Puppet provisioner always succeeds

I've been trying to diagnose an issue with [Packer](https://www.packer.io)'s [Puppet (Masterless) provisioner](https://www.packer.io/docs/provisioners/puppet-masterless.html) but the problem is that when puppet fails, Packer aborts and the AMI isn't saved. I found a solution which is to make it always succeed (as far as Packer is concerned), so that you can start an instance from the AMI, ssh in, and see what went wrong.

The solution is to add an `execute_command` property to the `puppet-masterless` provisioner, like this:

```json
{
  "type": "puppet-masterless",
  "manifest_file": "puppet/manifests/default.pp",
  "hiera_config_path": "puppet/hiera.yaml",
  "module_paths": [
     "puppet/modules",
  ],
  "extra_arguments": [
    "--debug"
  ],
  "execute_command":"cd {{.WorkingDir}} && {{.FacterVars}}{{if .Sudo}} sudo -E {{end}}puppet apply --verbose --modulepath='{{.ModulePath}}' {{if ne .HieraConfigPath \"\"}}--hiera_config='{{.HieraConfigPath}}' {{end}} {{if ne .ManifestDir \"\"}}--manifestdir='{{.ManifestDir}}' {{end}} --detailed-exitcodes {{.ManifestFile}}; /bin/true"
}
```

This is basically a copy of the build-in command (at least according to the Packer docs), with `; /bin/true` appended. I've also added `--debug` to `extra_arguments` to further aid debugging efforts.