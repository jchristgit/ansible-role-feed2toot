# feed2toot Ansible role

This role deploys the excellent
[feed2toot](https://gitlab.com/chaica/feed2toot) on a Debian 11 server. Other
distributions may work as well, if they use systemd and the `feed2toot` and
`python3-pexpect` features are available.

It performs a full automatic setup of feed2toot bots, including automating the
`register_feed2toot_app` script.

## Configuration

- `feed2toot_timer_schedule` (`str`): Represents the systemd timer specification
  to put into the final `OnCalendar`. The default, `hourly`, will result in your
  bot(s) checking for new posts every hour.

- `feed2toot_configurations` (`map[str, map[str, str]]`): A mapping of bot names
  to their configuration. The bot names can be anything systemd accepts in a
  unit name, and the configuration contains the following:

  - `mastodon_url` (`str`): The URL of the Mastodon instance where content
    should be reposted.

  - `rss_uri` (`str`): The URL of the RSS feed where content should be fetched
    from.

  The following two variables are optional. If set, this role will automatically
  run the `register_feed2toot_app` script to fetch client credentials for the
  application. If you do not use this, please put your credentials into
  `/etc/opt/feed2toot/$BOT_NAME/{user,client}.cred` yourself. Usage of
  [ansible-vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html)
  is recommended.

  - `mastodon_email`: The email address of the bot account that will be
    reposting RSS entries.

  - `mastodon_password`: The password to log in with.


## Example

The following configuration fetches blog posts from my website daily, and
reposts them to mastodon.social:

```yml
feed2toot_timer_schedule: daily
feed2toot_configurations:
  jchrist:
    mastodon_url: https://mastodon.social
    rss_uri: https://jchri.st/blog.rss
    mastodon_email: mybot@hello.me
    mastodon_password: "{{ vault_feed2toot_configurations.jchrist.mastodon_password }}"
```

This configuration will start and enable a timer named
`feed2toot@jchrist.timer`, which in turn will start the
`feed2toot@jchrist.service` whenever it elapses. Timer status can be queried
with `systemctl list-timers`, and logs can be read via `journalctl -xeu
feed2toot@jchrist.service`.


## Details

This role deploys files to the following locations:

- `/etc/opt/feed2toot/$BOTNAME`

The following directories are added and managed by the systemd service:

- `/var/cache/private/feed2toot/$BOTNAME` for cached RSS feed information -
  mainly to prevent reposts.

- `/run/feed2toot/$BOTNAME` for the lockfile.

The user is dynamically allocated via the `DynamicUser` directive. The script is
slightly sandboxed. If this causes issues for you, please let me know.


<!-- vim: set textwidth=80 sw=2 ts=2: -->
