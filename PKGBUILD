# Maintainer: p2k <me@p2k-network.org>
pkgname=wakfu
pkgver=1.0
pkgrel=2
pkgdesc="A turn-based tactical Massively Multiplayer Online Role-playing Game (MMORPG) written in Java/OpenGL."
url="http://www.wakfu.com/"
license="custom"
arch=('i686' 'x86_64')
source=(http://dl.ak.ankama.com/games/wakfu/client/linux/Wakfu_unix.sh UpLauncher.arch.sh)
if [ "$CARCH" == "x86_64" ];then
  depends=('lib32-gtk2' 'lib32-freetype2' 'lib32-libxt' 'lib32-libjpeg6'
           'lib32-libpng12' 'lib32-gvfs' 'lib32-libxslt' 'lib32-alsa-lib'
           'lib32-libtxc_dxtn' 'lib32-dbus-glib')
else
  depends=('gtk2' 'freetype2' 'libxt' 'libjpeg6'
           'libpng12' 'gvfs' 'libxslt' 'alsa-lib'
           'libtxc_dxtn' 'dbus-glib')
fi
makedepends=('unzip')
install=wakfu.install
md5sums=('21e024b1de364ad74ae2e8feceea4069'
         'b8589ed6a1f4f52e7e83be538a9c8284')

build() {
  cd "$srcdir"

  # We have to duplicate some functionality of install4j in order to
  # fit wakfu in an Arch Linux package and prevent it from messing
  # around in the user's home directory on packaging.

  msg2 "Extracting installer..."

  tail_cmd=`egrep -o -a -m 1 "tail -c [0-9]+" Wakfu_unix.sh`
  [ "$tail_cmd" = "" ] && return 1
  $tail_cmd Wakfu_unix.sh > sfx_archive.tar.gz

  [ -d wakfu_install ] && rm -rf wakfu_install

  mkdir wakfu_install
  cd wakfu_install
  tar xzf ../sfx_archive.tar.gz

  msg2 "Unpacking bundled JRE..."

  mkdir jre
  cd jre
  tar xzf ../jre.tar.gz
  INSTALL4J_JAVA_HOME_OVERRIDE=`pwd`

  msg2 "Preparing bundled JRE ..."
  jar_files="lib/rt.jar lib/charsets.jar lib/plugin.jar lib/deploy.jar lib/ext/localedata.jar lib/jsse.jar"
  for jar_file in $jar_files
  do
    if [ -f "${jar_file}.pack" ]; then
      bin/unpack200 -r ${jar_file}.pack $jar_file
      if [ $? -ne 0 ]; then
        echo "Error unpacking jar files. The architecture or bitness (32/64)"
        echo "of the bundled JVM might not match your machine."
        return 1
      fi
    fi
  done
  
  cd ..

  msg2 "Running installer..."

  INSTALL4J_NO_DB=yes
  INSTALL4J_TEMP=`pwd`
  
  cat > Wakfu_unix.varfile <<EOF
#install4j response file for WakfuLauncher 2
sys.languageId=en
sys.installationDir=$pkgdir/opt/wakfu
sys.programGroup.enabled\$Boolean=false
sys.programGroup.linkDir=/usr/local/bin
sys.programGroup.allUsers\$Boolean=false
sys.programGroup.name=WakfuLauncher
EOF

  sh ../Wakfu_unix.sh __i4j_lang_restart -q -varfile "$INSTALL4J_TEMP/Wakfu_unix.varfile" || return 1

  msg2 "Setting up paths and cleaning up..."

  cat > "$pkgdir/opt/wakfu/.install4j/install.prop" <<EOF
launcher0=/opt/wakfu/Wakfu
languageId=en
EOF

  echo "/opt/wakfu/jre" > "$pkgdir/opt/wakfu/.install4j/inst_jre.cfg"

  pattern=`sed 's/\\//\\\\\\//g' <<< "$pkgdir"`
  pattern="s/$pattern//g"
  sed -i "$pattern" "$pkgdir/opt/wakfu/.install4j/files.log"
  sed -i "$pattern" "$pkgdir/opt/wakfu/.install4j/response.varfile"

  rm -rf "jre/.systemPrefs" "$pkgdir/opt/wakfu/.install4j/installation.log" "$pkgdir/opt/wakfu/uninstall"
}

package() {
  cd "$srcdir/wakfu_install"
  install_dir="$pkgdir/opt/wakfu"

  msg2 "Installing bundled JRE..."
  cp -R jre "$install_dir/jre"

  msg2 "Installing launcher script..."
  install "$srcdir/UpLauncher.arch.sh" "$install_dir/UpLauncher.arch.sh"
  mkdir -p "$pkgdir/usr/bin"
  ln -s /opt/wakfu/UpLauncher.arch.sh "$pkgdir/usr/bin/wakfu"

  msg2 "Installing menu icon..."
  mkdir -p "$pkgdir/usr/share/applications"
  cat > "$pkgdir/usr/share/applications/Wakfu.desktop" <<EOF
[Desktop Entry]
Encoding=UTF-8
Type=Application
Name=Wakfu
GenericName=Wakfu
Comment=Wakfu
Icon=/opt/wakfu/.install4j/Wakfu.png
Exec=/opt/wakfu/UpLauncher.arch.sh
Path=/opt/wakfu/
Categories=Game;RolePlaying;Ankama-Games
EOF
}
# vim:set ts=2 sw=2 et:
