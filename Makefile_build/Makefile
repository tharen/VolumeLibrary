#directories
SOURCEDIR = ./src
MODDIR = $(SOURCEDIR)/mod
BUILDDIR = $(SOURCEDIR)/build
BINDIR = $(SOURCEDIR)/bin

#compiler
FC = gfortran

#compile flags; moved down into rules
FCFLAGS = 

#link flags
FLFLAGS = -shared -fPIC 

#DLL name
PROGRAM = volumelibrary

#Phony target- not sure if needed?
.PHONY: all clean

##source files, modules and objects

INCLUDES_SRC = $(wildcard $(SOURCEDIR)/*.inc $(SOURCEDIR)/*.INC)
INCLUDES_DEST = $(patsubst $(SOURCEDIR)/%,$(BUILDDIR)/%, $(INCLUDES_SRC))

#list of modules for reference: charmod.f clkcoef_mod.f debug_mod.f mrules_mod.f r8dib.f r8vlist.f volinput_mod.f

MODULES = $(wildcard $(MODDIR)/*.f)
MODULE_OBJECTS = $(patsubst $(MODDIR)/%.f,$(BUILDDIR)/%.o, $(MODULES))

SRCS = $(wildcard $(SOURCEDIR)/*.f $(SOURCEDIR)/*.F)
SRCS_SORTED = $(sort $(wildcard $(SOURCEDIR)/*.f $(SOURCEDIR)/*.F))

#string substitution to generate object names
OBJECTS = $(patsubst $(SOURCEDIR)/%.f,$(BUILDDIR)/%.o, $(SRCS))
OBJECTS_SORTED = $(patsubst $(SOURCEDIR)/%.f,$(BUILDDIR)/%.o, $(SRCS_SORTED))
	
##Make rules

#rule for modules - compile modules and direct .mod output to build folder with -J flag

 $(BUILDDIR)%.o : $(MODDIR)%.f
	$(FC) -c $< -J$(BUILDDIR) -o $@

#rules for include files (upper and lowercase extension), copy them from source directory to build directory
 $(BUILDDIR)%.inc:$(SOURCEDIR)%.inc
	cp  $< $@

 $(BUILDDIR)%.INC:$(SOURCEDIR)%.INC
	cp  $< $@
	
#rule for source files - compile, look for modules when compiling in build directory with -I flag
	
 $(BUILDDIR)/%.o : $(SOURCEDIR)/%.f
	$(FC) -c $< -I$(BUILDDIR) -o $@
	
all: $(INCLUDES_DEST) $(MODULE_OBJECTS) $(OBJECTS_SORTED)
	
$(BINDIR)/$(PROGRAM): $(PROGRAM)%.o
	$(FC) $(FLFLAGS)$^ -I$(BUILDDIR) -o $@
