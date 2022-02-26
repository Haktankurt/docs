# script to post commit messages to irc
# invoke from the 'update' hook with
# /srv/git/bin/xiphbot-notify.sh <refnam> <oldrev> <newrev>

refname="$1"
oldrev="$2"
newrev="$3"

# prefix to strip from the full repository path
prefix=/srv/git/repositories/

# notification script
UDP='/usr/bin/python /path/to/udp.py'
#UDP='echo -e'

# figure out our repository name
repopath=$(cd ${GIT_DIR} && pwd)
repo=${repopath#${prefix}}
repo=${repo#/var/www/git.xiph.org/}

tags=''
if [ ! -z $(echo ${repo} | grep opus) ]; then
  tags="${tags} #opus"
fi
if [ ! -z $(echo ${repo} | grep theora) ]; then
  tags="${tags} #theora"
fi
if [ ! -z $(echo ${repo} | grep daala) ]; then
  tags="${tags} #daala"
fi
if [ ! -z $(echo ${repo} | egrep 'ice|shout') ]; then
  tags="${tags} #icecast"
fi

# <http://www.mirc.com/colors.html>
COLOR_ESCAPE=`echo -ne "\003"`
RESET_COLOR=$COLOR_ESCAPE
ORANGE=${COLOR_ESCAPE}7
GREEN=${COLOR_ESCAPE}3
DARKBLUE=${COLOR_ESCAPE}2
BLUE=${COLOR_ESCAPE}12

# loop over the commits, posting a log msg for each one
for rev in $(git rev-list ${oldrev}..${newrev} | tac); do
  ${UDP} "$(git log --abbrev=12 -n 1 --format="$ORANGE$(basename ${repo%.git})$RESET_COLOR $BLUE${refname#refs/heads/}$RESET_COLOR $GREEN%an$DARKBLUE%d$RESET_COLOR %s https://git.xiph.org/?p=${repo};h=%h;a=commitdiff" ${rev}) ${tags}"
done
