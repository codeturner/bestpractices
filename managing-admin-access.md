# Managing Admin Access

## Enabling sudo to a non-admin user

Every user in the `admin` group can use sudo. To grant non-admin users the ability to use sudo, an admin will need to edit the sudoers file.

```sh
$ sudo visudo
Running command elevated
```

Then, add the user to the list of sudoers by adding a line such as this:

```txt
targetusername ALL=(ALL) ALL
```

## Adding/removing a user to the admin group from the command line

To add:

```sh
$ sudo dseditgroup -o edit -a targetusername -t user admin
$ sudo dseditgroup -o checkmember -m targetusername admin
yes targetusername is a member of admin
```

To remove:

```sh
$ sudo dseditgroup -o edit -d targetusername -t user admin
$ sudo dseditgroup -o checkmember -m targetusername admin
no targetusername is NOT a member of admin
```

To understand what these commands actually do, refer to this helpful blog post:

<https://travellingtechguy.blog/managing-and-reporting-unauthorised-admin-account-creation/>
