# ha-fly-tailscale

This project enables to deploy a [Home Assistant](https://www.home-assistant.io/) specific reverse proxy in the [fly.io](https://fly.io) cloud.
The reverse proxy will communicate with the local instance through a [Tailscale](https://tailscale.com/) VPN.

## Install flyctl

[Install flyctl](https://fly.io/docs/getting-started/installing-flyctl/)

- Get flyctrl util (linux)
`curl -L https://fly.io/install.sh | sh`

- Get flyctrl util (windows)
`iwr https://fly.io/install.ps1 -useb | iex`

- login
`flyctl auth login`

## Tailscale

- Create a Tailscale account if you don't have one, already
- Install the Tailscale client on your HA instance. If you are using a supervised HA install or HAssIO, just install the [Tailscale addon](https://github.com/hassio-addons/addon-tailscale)
- [Create an one-off key](https://login.tailscale.com/admin/settings/authkeys)

## Adjust fly.toml

Adjust `app = ` to a host name of your choice. It will be the hostname you'll use to access HA from the internet, e.g. `https://<my_app_id>.fly.dev`

**It must be unique across all fly, so be creative!**

```ini
  TAILSCALE_HOSTNAME = "fly-tailscale-nginx"              # This will be the host name registered in Tailscale
  TAILSCALE_HA_SCHEME = "http"                            # The HA scheme, http or https
  TAILSCALE_HA_HOST = "<ha_taiscale_ip>"                  # The Tailscale address of your HA instance
  TAILSCALE_HA_PORT = "8123"                              # The HA port
  TAILSCALE_AUTHKEY = "<the_tailscale_one-off_key>"       # The Tailscale one-off key you created earlier
```

## Create a Fly application with the util

- `flyctl apps create foobar-homeassistant` ("foobar-homeassistant" will be the name of your application in Fly)

## Deploying the application

### You have docker installed

Easy peasy. Just do `flyctl -a foobar-homeassistant deploy` to deploy the application on Fly.  
After a little while, the application will just be there, ready for usage

### You **don't** have docker installed

A bit trickier, here. Obviously, you could install it, but it might be involving, especially on Windows.
The alternative route is to let Github do the job.

- Fork this repository: [ha-fly-tailscale-base](https://github.com/koying/ha-fly-tailscale-base)
- Get a Fly API token: `flyctl auth token`
- Insert this token under the name `FLY_API_TOKEN` in github, in Settings-Secrets-New Repository secret
- Enable Worflow on your fork: Actions-"I understand my workflows, go ahead and enable them"
- Edit the `fly.toml` file, possibly in github itself, as described above.
- Commit your changes.
- Voilà! Github should handle the deployment of your Fly app via Actions.

**SECURITY NOTE:** Unfortunately, it's not possible to make a fork of a public repository private in github, so you are now leaking private information in a public repository. **Delete the repository from github as soon as the deployment is successful!**
An alternative, more secured but more involving, way of working is to duplicate the repo and create a private copy, instead. See, e.g., [GitHub: How to make a fork of public repository private?](https://medium.com/@bilalbayasut/github-how-to-make-a-fork-of-public-repository-private-6ee8cacaf9d3)

## Adjust configuration.yaml in HA to accept the remote proxy you just created

- In the Tailscale dashboard, note the IP address that was assigned to your Fly application
- In the "3-dots" menu, select "Disable key expiry'.
- Add the IP to your HA "trusted_proxies".

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies: 
    - "<tailscale_foobar-homeassistant_ip>"
```

## All done!

Just go to `https://<my_app_id>.fly.dev` to access your HA instance from anywhere!
