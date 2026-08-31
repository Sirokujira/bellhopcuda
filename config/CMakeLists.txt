# bellhopcxx / bellhopcuda - C++/CUDA port of BELLHOP / BELLHOP3D underwater acoustics simulator
# Copyright (C) 2021-2023 The Regents of the University of California
# Marine Physical Lab at Scripps Oceanography, c/o Jules Jaffe, jjaffe@ucsd.edu
# Based on BELLHOP / BELLHOP3D, which is Copyright (C) 1983-2022 Michael B. Porter
# 
# This program is free software: you can redistribute it and/or modify it under
# the terms of the GNU General Public License as published by the Free Software
# Foundation, either version 3 of the License, or (at your option) any later
# version.
# 
# This program is distributed in the hope that it will be useful, but WITHOUT ANY
# WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
# PARTICULAR PURPOSE. See the GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License along with
# this program. If not, see <https://www.gnu.org/licenses/>.

# Ensure the clang-format git hook is installed.
set(copy_hook 0)
if(NOT EXISTS "${PROJECT_SOURCE_DIR}/.git/hooks/pre-commit")
    message(WARNING "clang-format pre-commit hook not found, installing")
    set(copy_hook 1)
else()
    file(MD5 "${PROJECT_SOURCE_DIR}/.git/hooks/pre-commit" precommit_md5)
    file(MD5 "${PROJECT_SOURCE_DIR}/clang-format.hook" curhook_md5)
    # STREQUAL, not EQUAL: EQUAL is a numeric comparison, and MD5 hashes are hex
    # strings. It misfires both ways -- two identical hashes beginning with a hex
    # letter compare as different (warn and rewrite on every configure), while two
    # different hashes sharing a numeric prefix compare as equal (stale hook kept).
    if(NOT precommit_md5 STREQUAL curhook_md5)
        message(WARNING "clang-format pre-commit hook is outdated or a different "
        "pre-commit hook was installed, overwriting with current clang-format hook")
        set(copy_hook 1)
    endif()
endif()
# A hook installed before the permissions fix below has a matching MD5 but no
# executable bit, so the check above leaves it in place and git keeps ignoring
# it. Test the bit separately to repair those clones. UNIX only: Windows has no
# executable bit and git there runs hooks regardless.
if(NOT copy_hook AND UNIX)
    execute_process(
        COMMAND test -x "${PROJECT_SOURCE_DIR}/.git/hooks/pre-commit"
        RESULT_VARIABLE hook_exec_test
    )
    if(NOT hook_exec_test EQUAL 0)
        message(WARNING "clang-format pre-commit hook is not executable, so git "
        "ignores it; reinstalling with the executable bit set")
        set(copy_hook 1)
    endif()
endif()
if(copy_hook)
    # Git silently ignores a hook that is not executable, so set the bit here
    # rather than relying on the mode file(COPY) carries over from the source.
    # FILE_PERMISSIONS keeps this working on CMake 3.15 (file(CHMOD) needs 3.19).
    file(COPY "${PROJECT_SOURCE_DIR}/clang-format.hook"
        DESTINATION "${PROJECT_SOURCE_DIR}/.git/hooks"
        FILE_PERMISSIONS
            OWNER_READ OWNER_WRITE OWNER_EXECUTE
            GROUP_READ GROUP_EXECUTE
            WORLD_READ WORLD_EXECUTE
    )
    file(RENAME "${PROJECT_SOURCE_DIR}/.git/hooks/clang-format.hook"
        "${PROJECT_SOURCE_DIR}/.git/hooks/pre-commit"
    )
endif()

add_subdirectory(cxx)

if(BHC_ENABLE_CUDA AND NOT(APPLE OR DEFINED ENV{BHC_NO_CUDA}))
    add_subdirectory(cuda)
endif()

if(BHC_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()
