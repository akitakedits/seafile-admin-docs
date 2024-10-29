# Roles and Permissions Support

You can add/edit roles and permission for users. A role is just a group of users with some pre-defined permissions, you can toggle user roles in user list page at admin panel.

`role_quota` is used to set quota for a certain role of users. For example, we can set the quota of employee to 100G by adding `'role_quota': '100g'`, and leave other role of users to the default quota.

`can_add_public_repo` is to set whether a role can create a public library, default is "False".

!!! warning "The `can_add_public_repo` option will not take effect if you configure global `CLOUD_MODE = True`"

The `storage_ids` permission is used for assigning storage backends to users with specific role. More details can be found in [multiple storage backends](../setup/setup_with_multiple_storage_backends.md).

Since version 10.0, `upload_rate_limit` and `download_rate_limit` are added to limit upload and download speed for users with different roles. **After configured the rate limit, run the following command in the `seafile-server-latest` directory to make the configuration take effect**:

```
./seahub.sh python-env python3 seahub/manage.py set_user_role_upload_download_rate_limit
```

Since version 11.0.9 pro, `can_share_repo` is added to limit users' ability to share a library.

Seafile comes with two build-in roles `default` and `guest`, a default user is a normal user with permissions as followings:

```
    'default': {
        'can_add_repo': True,
        'can_share_repo': True,
        'can_add_group': True,
        'can_view_org': True,
        'can_add_public_repo': False,
        'can_use_global_address_book': True,
        'can_generate_share_link': True,
        'can_generate_upload_link': True,
        'can_send_share_link_mail': True,
        'can_invite_guest': False,
        'can_connect_with_android_clients': True,
        'can_connect_with_ios_clients': True,
        'can_connect_with_desktop_clients': True,
        'storage_ids': [],
        'role_quota': '',
        'can_publish_repo': True,
        'upload_rate_limit': 0,  # unit: kb/s
        'download_rate_limit': 0,
    },
```

While a guest user can only read files/folders in the system, here are the permissions for a guest user:
```
    'guest': {
        'can_add_repo': False,
        'can_share_repo': False,
        'can_add_group': False,
        'can_view_org': False,
        'can_add_public_repo': False,
        'can_use_global_address_book': False,
        'can_generate_share_link': False,
        'can_generate_upload_link': False,
        'can_send_share_link_mail': False,
        'can_invite_guest': False,
        'can_connect_with_android_clients': False,
        'can_connect_with_ios_clients': False,
        'can_connect_with_desktop_clients': False,
        'storage_ids': [],
        'role_quota': '',
        'can_publish_repo': False,
        'upload_rate_limit': 0,
        'download_rate_limit': 0,
    },
```

## Edit build-in roles

If you want to edit the permissions of build-in roles, e.g. default users can invite guest, guest users can view repos in organization, you can add following lines to `seahub_settings.py` with corresponding permissions set to `True`.

```
ENABLED_ROLE_PERMISSIONS = {
    'default': {
        'can_add_repo': True,
        'can_share_repo': True,
        'can_add_group': True,
        'can_view_org': True,
        'can_add_public_repo': False,
        'can_use_global_address_book': True,
        'can_generate_share_link': True,
        'can_generate_upload_link': True,
        'can_send_share_link_mail': True,
        'can_invite_guest': True,
        'can_connect_with_android_clients': True,
        'can_connect_with_ios_clients': True,
        'can_connect_with_desktop_clients': True,
        'storage_ids': [],
        'role_quota': '',
        'can_publish_repo': True,
        'upload_rate_limit': 2000,  # unit: kb/s
        'download_rate_limit': 4000,
    },
    'guest': {
        'can_add_repo': False,
        'can_share_repo': False,
        'can_add_group': False,
        'can_view_org': True,
        'can_add_public_repo': False,
        'can_use_global_address_book': False,
        'can_generate_share_link': False,
        'can_generate_upload_link': False,
        'can_send_share_link_mail': False,
        'can_invite_guest': False,
        'can_connect_with_android_clients': False,
        'can_connect_with_ios_clients': False,
        'can_connect_with_desktop_clients': False,
        'storage_ids': [],
        'role_quota': '',
        'can_publish_repo': False,
        'upload_rate_limit': 100,
        'download_rate_limit': 200,
    }
}
```

### More about guest invitation feature

An user who has `can_invite_guest` permission can invite people outside of the organization as guest.

In order to use this feature, in addition to granting `can_invite_guest` permission to the user, add the  following line to `seahub_settings.py`,

```
ENABLE_GUEST_INVITATION = True

# invitation expire time
INVITATIONS_TOKEN_AGE = 72 # hours
```

After restarting, users who have `can_invite_guest` permission will see "Invite People" section at sidebar of home page.

Users can invite a guest user by providing his/her email address, system will email the invite link to the user.

!!! tip "Tip"
    If you want to block certain email addresses for the invitation, you can define a blacklist, e.g.

    ```
    INVITATION_ACCEPTER_BLACKLIST = ["a@a.com", "*@a-a-a.com", r".*@(foo|bar).com", ]
    ```

After that, email address "a@a.com", any email address ends with "@a-a-a.com" and any email address ends with "@foo.com" or "@bar.com" will not be allowed.


## Add custom roles

If you want to add a new role and assign some users with this role, e.g. new role `employee` can invite guest and can create public library and have all other permissions a default user has, you can add following lines to `seahub_settings.py`

```
ENABLED_ROLE_PERMISSIONS = {
    'default': {
        'can_add_repo': True,
        'can_share_repo': True,
        'can_add_group': True,
        'can_view_org': True,
        'can_add_public_repo': False,
        'can_use_global_address_book': True,
        'can_generate_share_link': True,
        'can_generate_upload_link': True,
        'can_send_share_link_mail': True,
        'can_invite_guest': False,
        'can_connect_with_android_clients': True,
        'can_connect_with_ios_clients': True,
        'can_connect_with_desktop_clients': True,
        'storage_ids': [],
        'role_quota': '',
        'can_publish_repo': True,
        'upload_rate_limit': 2000,  # unit: kb/s
        'download_rate_limit': 4000,
    },
    'guest': {
        'can_add_repo': False,
        'can_share_repo': False,
        'can_add_group': False,
        'can_view_org': False,
        'can_add_public_repo': False,
        'can_use_global_address_book': False,
        'can_generate_share_link': False,
        'can_generate_upload_link': False,
        'can_send_share_link_mail': False,
        'can_invite_guest': False,
        'can_connect_with_android_clients': False,
        'can_connect_with_ios_clients': False,
        'can_connect_with_desktop_clients': False,
        'storage_ids': [],
        'role_quota': '',
        'can_publish_repo': False,
        'upload_rate_limit': 100,
        'download_rate_limit': 200,
    },
    'employee': {
        'can_add_repo': True,
        'can_share_repo': True,
        'can_add_group': True,
        'can_view_org': True,
        'can_add_public_repo': True,
        'can_use_global_address_book': True,
        'can_generate_share_link': True,
        'can_generate_upload_link': True,
        'can_send_share_link_mail': True,
        'can_invite_guest': True,
        'can_connect_with_android_clients': True,
        'can_connect_with_ios_clients': True,
        'can_connect_with_desktop_clients': True,
        'storage_ids': [],
        'role_quota': '',
        'can_publish_repo': True,
        'upload_rate_limit': 500,
        'download_rate_limit': 800,
    },
}
```
