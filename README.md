# headscale-configs

This repository contains configurations which are more or less ready to go for you to use.

The stack...
* [Docker](https://www.docker.com)
* [Headscale](https://headscale.net/)
* [Headplane (Admin UI)](https://github.com/tale/headplane/)
* [Caddy (Reverse Proxy)](https://caddyserver.com/)
* [Authelia (OIDC Authentication)](https://www.authelia.com/)

I am assuming that you already have Authelia up and running in your environment. If you don't go [check their docs](https://www.authelia.com/overview/prologue/introduction/) to learn more. I also am assuming you have Docker installed on a VPS with a public IP, if you don't here are the [Debian docs](https://docs.docker.com/engine/install/debian/) (I use Debian, you'll have to find the docs for your OS if it's not Debian).

If you don't have a VPS with a public IP check out [BuyVM](https://www.buyvm.net) (*#notasponsor*), I'm in no way affilated with them, I'm just a customer who likes their service.

## Authelia

There is one file in the repo associated with Authelia and that is [authelia-configuration.yaml](/authelia-configuration.yaml).

The configuration file is two snippets, one for OIDC authorization policies and the other for client configuration for Headscale and Headplane. In my environment I have the two seperated so that I can have users who can use Headscale but not auth to Headplane and vice versa if I wanted. 

For Headscale to support this it does require the 'groups' scope in it's OIDC client configuration, without it you cannot use the allowed_groups field in the Headscale config. The authorization policy and the Headscale "allowed_groups" is redundant but oh well.

## Headscale/Headplane

Start with the [headscale-configuration.yaml](/headscale-configuration.yaml) file as you need this to even start headscale. Check through the entire file if you're going to use it, I literally just took mine and sanitized it, you'll want to change stuff.

Once you have that done you can move onto the docker piece. Before you do anything with any docker files, go onto the docker host which will be hosting this and create a caddy network completely seperate from any compose file.

```bash
docker network create caddy
```

This way your network doesn't disappear and/or nothing happens to it when you're messing with the compose files. Now that you've done that, you can start with the [headscale-headplane-docker-compose.yaml](/headscale-headplane-docker-compose.yaml) file.

You'll want to deploy only headscale first, so remove the headplane stuff for now and just deploy headscale. Once you have headscale deployed run...

```bash
docker exec headscale headscale apikeys create
```

This command creates the `ROOT_API_KEY` which we need in the headplane docker config. Now that you have that you can add the headplane service back to the compose file and update it as needed. Remember to update the `COOKIE_SECRET` and `ROOT_API_KEY` as needed. Then fill in the OIDC with your Authelia information.

## Caddy

We're using Caddy as a reverse-proxy for both Headscale and Headplane. We'll start with the [Caddyfile](/Caddyfile). All you should really have to do here is replace `example.com` with your domain. Make sure you have an A record setup in your DNS pointing `hs.yourdomain.com` to the public IP of your Docker host.

Once you have the Caddyfile you can then take the [caddy-docker-compose.yaml](/caddy-docker-compose.yaml) and deploy it as is.

## Next Steps

Assuming you've made the appropriate changes to your files, added the proper DNS records, and started the docker containers, everything should come up. You are able to confirm if this is working by browsing to `https://hs.yourdomain.com/admin` for Headplane, you should hit your Authelia instance right away. Then also check to make sure `https://hs.yourdomain.com/windows` works. It should give you instructions on how to setup a Windows client.

If both of those work you're in business, if not...you might have missed something above. Be sure to check your container logs (headplane, headscale and caddy) as well as your Authelia instance logs if you encounter any errors. They're pretty valuable and the couple errors I encounted were meaningful.

I also suggest setting up iptables and iptables-persistant with whatever firewall rules you need, friends don't let friends have unprotected publicly facing hosts :stuck_out_tongue_winking_eye:

## Disclaimer

The instructions provided in this repository are for informational purposes only. While I have made every effort to ensure the accuracy and reliability of the information provided, I assume no responsibility for errors, omissions, or damages resulting from the use of these instructions. Use the information at your own risk. Always perform thorough testing and backups before deploying any changes to your environment.