subdir('builder')
subdir('pki')
subdir('remotes.d')
subdir('bash-completion')

if get_option('tests') and get_option('daemon')
  subdir('tests')
  subdir('installed-tests')
endif

install_data(['daemon.conf'],
  install_dir : join_paths(sysconfdir, 'fwupd')
)

install_data(['org.freedesktop.fwupd.metainfo.xml'],
  install_dir: join_paths(datadir, 'metainfo')
)

install_data(['org.freedesktop.fwupd.conf'],
  install_dir : join_paths(sysconfdir, 'dbus-1', 'system.d')
)

install_data(['metadata.xml'],
  install_dir : join_paths(datadir, 'fwupd', 'remotes.d', 'fwupd')
)

if get_option('daemon')
  install_data(['90-fwupd-devices.rules'],
    install_dir : join_paths(udevdir, 'rules.d')
  )
endif

if get_option('systemd')
  con2 = configuration_data()
  con2.set('libexecdir', libexecdir)
  con2.set('bindir', bindir)
  con2.set('datadir', datadir)

  rw_directories = []
  rw_directories += join_paths (localstatedir, 'lib', 'fwupd')
  rw_directories += join_paths (default_sysconfdir, 'fwupd', 'remotes.d')
  if get_option('plugin_uefi')
    rw_directories += ['-/boot/efi', '-/efi', '-/boot/EFI']
  endif

  dynamic_options = []
  if systemd.version().version_compare('>= 232')
    dynamic_options += 'ProtectControlGroups=yes'
    dynamic_options += 'ProtectKernelModules=yes'
  endif
  if systemd.version().version_compare('>= 231')
    dynamic_options += 'RestrictRealtime=yes'
    dynamic_options += 'MemoryDenyWriteExecute=yes'
    dynamic_options += ['ReadWritePaths=' + ' '.join(rw_directories)]
  else
    dynamic_options += ['ReadWriteDirectories=' + ' '.join(rw_directories)]
  endif
  con2.set('dynamic_options', '\n'.join(dynamic_options))

  # replace @libexecdir@
  configure_file(
    input : 'org.freedesktop.fwupd.service.in',
    output : 'org.freedesktop.fwupd.service',
    configuration : con2,
    install: true,
    install_dir: join_paths(datadir,
                            'dbus-1',
                            'system-services'),
  )

  # replace @bindir@
  configure_file(
    input : 'fwupd-offline-update.service.in',
    output : 'fwupd-offline-update.service',
    configuration : con2,
    install: true,
    install_dir: systemdunitdir,
  )

  # replace @dynamic_options@
  configure_file(
    input : 'fwupd.service.in',
    output : 'fwupd.service',
    configuration : con2,
    install: true,
    install_dir: systemdunitdir,
  )
endif
