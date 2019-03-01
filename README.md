### About This Service Package
Set configuration values in your own `.toml` file and override the default
configurations in `default.toml` like this:
```
hab config apply --remote-sup=<hostname> <service>.<group> $(date +%s) ./foo.toml
```
See https://www.habitat.sh/docs/using-habitat/#config-updates for more on
configuration updates.

Certificates will be stored at
```
/hab/svc/certbot/config/live/<domain name>/*.pem
```
which means that you can point configs for other services (like `core/nginx` or
`core/apache`) at these directories to load certificates. The usual caveats about
securing `/hab/svc/certbot/config` certainly apply here. You can also consider
backing up this directory.

#### AWS Credentials
The current version of this package supports only certificate verification using
Route53 DNS. You should set the appropriate `AWS_SECRET_ACCESS_KEY` and
`AWS_ACCESS_KEY_ID` environment variables for the runtime context of the service.
Here's an example of how `bixu/certbot` might be run as a systemd service with
AWS credentials embedded:
```
[Unit]
Description=Certbot
[Service]
Environment="HAB_AUTH_TOKEN=<some value>"
Environment="AWS_ACCESS_KEY_ID=<some value>"
Environment="AWS_SECRET_ACCESS_KEY=<some value>"
ExecStart=/bin/hab sup run bixu/certbot
KillMode=process
Restart=on-failure
[Install]
WantedBy=default.target
```
Future versions of the service may support other verification plugins as well as
other methods for handling secrets.

#### Caveats
At the moment, this package only supports Route53 DNS verification of domain
ownership. Since only a single instance of a Habitat package can be running at
any given time, it's recommended to use this service to register wildcard
LetsEncrypt certificates only, so you should configure the `domain` TOML
key to a value of `*.example.com`, as an example.
