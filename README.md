<h3 align="center">
	Discord Linked Roles OAuth2
</h3>
<h5 align="center">
  A basic wrapper for the Discord Linked Roles OAuth2 API.
</h5>

<p align="center">
    <img alt="Releases" src="https://img.shields.io/github/release/staciax/discord-linked-roles.svg?style=for-the-badge&logo=github&color=F2CDCD&logoColor=D9E0EE&labelColor=302D41"/></a>
	<a href="https://github.com/staciax/discord-linked-roles/stargazers"><img src="https://img.shields.io/github/stars/staciax/discord-linked-roles?colorA=363a4f&colorB=b7bdf8&style=for-the-badge"></a>
	<a href="https://github.com/staciax/discord-linked-roles/issues"><img src="https://img.shields.io/github/issues/staciax/discord-linked-roles?colorA=363a4f&colorB=f5a97f&style=for-the-badge"></a>
	<a href="https://github.com/staciax/discord-linked-roles/contributors"><img src="https://img.shields.io/github/contributors/staciax/discord-linked-roles?colorA=363a4f&colorB=a6da95&style=for-the-badge"></a>
</p>

<!-- https://github.com/catppuccin color style -->

## Installation
```sh
$ pip install -U linked-roles
```

## FastAPI Example:
```py
from fastapi import FastAPI
from fastapi.responses import RedirectResponse

import config
from linked_roles import LinkedRolesOAuth2, OAuth2Scopes, RolePlatform, UserNotFound

app = FastAPI(title='Linked Roles OAuth2')

client = LinkedRolesOAuth2(
    client_id=config.DISCORD_CLIENT_ID,
    client_secret=config.DISCORD_CLIENT_SECRET,
    redirect_uri=config.DISCORD_REDIRECT_URI,
    # token=config.DISCORD_TOKEN, # Optinal for Resgister
    scopes=(OAuth2Scopes.role_connection_write, OAuth2Scopes.identify),
    state=config.COOKIE_SECRET,
)

@app.on_event('startup')
async def startup():
    await client.start()

@app.on_event('shutdown')
async def shutdown():
    await client.close()

@app.get('/linked-role')
async def linked_roles():
    url = client.get_oauth_url()
    return RedirectResponse(url=url)

@app.get('/verified-role')
async def verified_role(code: str):

    # get tokens
    tokens = await client.get_oauth2_tokens(code)

    # get user
    user = await client.fetch_user(tokens=tokens)

    if user is None:
        raise UserNotFound('User not found')

    # set role platform
    platform = RolePlatform(name='VALORANT', username=str(user))

    # add metadata
    platform.add_metadata(key='matches', value=10)
    platform.add_metadata(key='winrate', value=20)
    platform.add_metadata(key='combat_score', value=30)

    # set role metadata
    await user.edit_role_metadata(platform=platform)

    return 'Role metadata set successfully. Please check your Discord profile.'
```

## Register Example:
```py
import asyncio

import config
from linked_roles import AppRoleConnectionMetadataRecordType as RoleMetadataType, LinkedRolesOAuth2, RoleMetadataRecord


async def main():

    client = LinkedRolesOAuth2(client_id=config.DISCORD_CLIENT_ID, token=config.DISCORD_TOKEN)

    async with client:

        records = (
            RoleMetadataRecord(
                key='matches',
                name='Matches',
                type=2,
            ),
            RoleMetadataRecord(
                key='winrate',
                name='Win Rate',
                type=RoleMetadataType.interger_greater_than_or_equal,  # Union Between int and RoleMetadataType
            ),
            RoleMetadataRecord(
                key='combat_score',
                name='Combat Score',
                description='Combat score this season', # description is optional (default: '...')
                type=RoleMetadataType.interger_greater_than_or_equal,
            )
        )

        records = await client.register_role_metadata(records=records)
        print(records)


if __name__ == '__main__':
    asyncio.run(main())

```
## Config Example:
```py
DISCORD_TOKEN = '<your bot token>'
DISCORD_CLIENT_ID = '<your client id>'
DISCORD_CLIENT_SECRET = '<your client secret>'
DISCORD_REDIRECT_URI = 'http://localhost:8000/verified-role'  # example redirect uri
COOKIE_SECRET = '<your cookie secret>'

# cookie secret can be generated with:
import uuid
>> uuid.uuid4().hex
```

## More Examples:
- fastapi more examples: [examples/fastapi](examples/fastapi_.py)

## TODO:
- [ ] Add more examples
- [ ] Add documentation
- [ ] Add database support (postgresql, sqlite, etc.) ?
- [ ] add more class error

<!-- code style, inspiration is discord.py -->
## Code Style Inspiration
<!-- https://github.com/Rapptz/discord.py -->
- [discord.py](https://github.com/Rapptz/discord.py)

## License
licensed under the [MIT license](LICENSE).
