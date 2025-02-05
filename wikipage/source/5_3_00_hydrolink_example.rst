Example of ``OpenWQ_hydrolink`` files
===============================================

The ``hydrolink`` files are the interface between the hydro-model and OpenWQ.

They serve two main purposes:

    * Convert data types between the hydro-model and OpenWQ (``COUPLER CODE``)
    * Make calls to OpenWQ's coupler functions

The ``hydrolink`` files provided below (``cpp`` and ``header``) are just an example of an implementation of the steps prescribed in section `"Steps to perform the coupling" <https://openwq.readthedocs.io/en/latest/5_3_0_Coupling_steps.html>`_.
This was an implementation developed for the `Cold Regions Hydrological Model (CRHM) <https://openwq.readthedocs.io/en/latest/5_3_0_CRHM.html>`_.
However, as previously mentioned, each model is different so these files have to be adapted accordingly.
Please also bear in mind that the ``COUPLER CODE`` here is very specific to this implementation and the structure of CRHM.
However, you should be able to clearly identify the calls to the different OpenWQ coupler functions.


OpenWQ_hydrolink.h
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: guess

    // This program, openWQ, is free software: you can redistribute it and/or modify
    // it under the terms of the GNU General Public License as published by
    // the Free Software Foundation, either version 3 of the License, or
    // (at your option) aNCOLS later version.
    //
    // This program is distributed in the hope that it will be useful,
    // but WITHOUT ANY WARRANTY; without even the implied warranty of
    // MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    // GNU General Public License for more details.

    // You should have received a copy of the GNU General Public License
    // along with this program.  If not, see <http://www.gnu.org/licenses/>.

    #ifndef OPENWQ_HYDROLINK_INCLUDED
    #define OPENWQ_HYDROLINK_INCLUDED

    // from CRHM
    #include "../vcl.h"
    //#pragma hdrstop

    #include "../WQ_CRHM.h"
    #include "../NewModules.h"
    #include "../GlobalDll.h"
    #include "../ClassModule.h"

    #include <math.h>
    #include <stdlib.h>

    //#pragma package(smart_init)
    using namespace CRHM;

    // for OpenWQ
    #include <iostream>
    #include <fstream>
    #include <armadillo>
    #include <string>

    #include "OpenWQ_couplercalls.h"
    #include "OpenWQ_global.h"
    #include "OpenWQ_readjson.h"
    #include "OpenWQ_initiate.h"
    #include "OpenWQ_chem.h"
    #include "OpenWQ_watertransp.h"
    #include "OpenWQ_sinksource.h"
    #include "OpenWQ_units.h"
    #include "OpenWQ_solver.h"
    #include "OpenWQ_output.h"


    class CLASSWQ_openwq : public ClassModule
    {

        public:

        CLASSWQ_openwq(std::string Name, std::string Version = "undefined", CRHM::LMODULE Lvl = CRHM::PROTO) : ClassModule(Name, Version, Lvl) {};

        // Variables from CRHM
        const float *hru_t; // has to be converted to soil temperatures.
        const float *hru_area; //
        const float *SWE;
        float *SWE_conc;
        float **SWE_conc_lay;
        const float *soil_runoff;
        float *soil_runoff_cWQ;
        float **soil_runoff_cWQ_lay;
        const float *soil_ssr;
        float *soil_ssr_conc;
        float **soil_ssr_conc_lay;
        const float *soil_lower;
        float *soil_lower_conc;
        float **soil_lower_conc_lay;
        const float *soil_rechr;
        float *conc_soil_rechr;
        float **conc_soil_rechr_lay;
        float *surfsoil_solub_mWQ;
        float **surfsoil_solub_mWQ_lay;
        float *conc_soil_lower;   // concentration of organic nitrogen *** from soilstate
        float **conc_soil_lower_lay;
        const float *soil_moist;
        float *Sd;
        float *Sd_conc;
        float **Sd_conc_lay;
        float *gw;
        float *gw_conc;
        float **gw_conc_lay;
        const float *soil_rechr_max;


        void decl(
            OpenWQ_couplercalls& OpenWQ_couplercalls,
            OpenWQ_hostModelconfig& OpenWQ_hostModelconfig,
            OpenWQ_json& OpenWQ_json,                    // create OpenWQ_json object
            OpenWQ_wqconfig& OpenWQ_wqconfig,            // create OpenWQ_wqconfig object
            OpenWQ_units& OpenWQ_units,                  // functions for unit conversion
            OpenWQ_readjson& OpenWQ_readjson,            // read json files
            OpenWQ_vars& OpenWQ_vars,
            OpenWQ_initiate& OpenWQ_initiate,            // initiate modules
            OpenWQ_watertransp& OpenWQ_watertransp,      // transport modules
            OpenWQ_chem& OpenWQ_chem,                   // biochemistry modules
            OpenWQ_sinksource& OpenWQ_sinksource,        // sink and source modules)
            OpenWQ_output& OpenWQ_output,                // output modules (needed for console/logfile)
            unsigned long num_hru);

        void run(
            OpenWQ_couplercalls& OpenWQ_couplercalls,
            OpenWQ_hostModelconfig& OpenWQ_hostModelconfig,
            OpenWQ_json& OpenWQ_json,                    // create OpenWQ_json object
            OpenWQ_wqconfig& OpenWQ_wqconfig,            // create OpenWQ_wqconfig object
            OpenWQ_units& OpenWQ_units,                  // functions for unit conversion
            OpenWQ_readjson& OpenWQ_readjson,            // read json files
            OpenWQ_vars& OpenWQ_vars,
            OpenWQ_initiate& OpenWQ_initiate,            // initiate modules
            OpenWQ_watertransp& OpenWQ_watertransp,      // transport modules
            OpenWQ_chem& OpenWQ_chem,                   // biochemistry modules
            OpenWQ_sinksource& OpenWQ_sinksource,        // sink and source modules)
            OpenWQ_solver& OpenWQ_solver,                // solver module
            OpenWQ_output& OpenWQ_output);               // output modules

    };

    #endif


OpenWQ_hydrolink.cpp
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: cpp

    // Copyright 2020-22, Diogo Costa, diogo.costa@usask.ca
    // This file is part of OpenWQ model.

    // This program, openWQ, is free software: you can redistribute it and/or modify
    // it under the terms of the GNU General Public License as published by
    // the Free Software Foundation, either version 3 of the License, or
    // (at your option) aNCOLS later version.
    //
    // This program is distributed in the hope that it will be useful,
    // but WITHOUT ANY WARRANTY; without even the implied warranty of
    // MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    // GNU General Public License for more details.

    // You should have received a copy of the GNU General Public License
    // along with this program.  If not, see <http://www.gnu.org/licenses/>.


    #include "OpenWQ_hydrolink.h"

    using namespace CRHM;

    //************************************************************************************************
    // Declare and initiate variables
    // function that calls coupler function: OpenWQ_couplercalls.InitialConfig
    //************************************************************************************************
    void CLASSWQ_openwq::decl(
        OpenWQ_couplercalls& OpenWQ_couplercalls,     // Class with all call from coupler
        OpenWQ_hostModelconfig& OpenWQ_hostModelconfig,
        OpenWQ_json& OpenWQ_json,                    // create OpenWQ_json object
        OpenWQ_wqconfig& OpenWQ_wqconfig,            // create OpenWQ_wqconfig object
        OpenWQ_units& OpenWQ_units,                  // functions for unit conversion
        OpenWQ_readjson& OpenWQ_readjson,            // read json files
        OpenWQ_vars& OpenWQ_vars,
        OpenWQ_initiate& OpenWQ_initiate,            // initiate modules
        OpenWQ_watertransp& OpenWQ_watertransp,      // transport modules
        OpenWQ_chem& OpenWQ_chem,                   // biochemistry modules
        OpenWQ_sinksource& OpenWQ_sinksource,        // sink and source modules)
        OpenWQ_output& OpenWQ_output,
        unsigned long nhru)
        {

        // Check if the call is from ::decl (allow to set OpenWQ_hostModelconfig)
        // or ::initbase (do not allow because OpenWQ_hostModelconfig has already been
        // defined)
        if (OpenWQ_hostModelconfig.HydroComp.size()==0){

            // #######################################
            // Characterize the Host model domain
            // Host model specific
            // #######################################
            OpenWQ_hostModelconfig.HydroComp.push_back(OpenWQ_hostModelconfig::hydroTuple(0,"SWE",nhru,1,1));
            OpenWQ_hostModelconfig.HydroComp.push_back(OpenWQ_hostModelconfig::hydroTuple(1,"RUNOFF",nhru,1,1));
            OpenWQ_hostModelconfig.HydroComp.push_back(OpenWQ_hostModelconfig::hydroTuple(2,"SSR",nhru,1,1));
            OpenWQ_hostModelconfig.HydroComp.push_back(OpenWQ_hostModelconfig::hydroTuple(3,"SD",nhru,1,1));
            OpenWQ_hostModelconfig.HydroComp.push_back(OpenWQ_hostModelconfig::hydroTuple(4,"SOIL_RECHR",nhru,1,1));
            OpenWQ_hostModelconfig.HydroComp.push_back(OpenWQ_hostModelconfig::hydroTuple(5,"SOIL_LOWER",nhru,1,1));
            OpenWQ_hostModelconfig.HydroComp.push_back(OpenWQ_hostModelconfig::hydroTuple(6,"SURFSOIL",nhru,1,1));
            OpenWQ_hostModelconfig.HydroComp.push_back(OpenWQ_hostModelconfig::hydroTuple(7,"GW",nhru,1,1));
            // (add other compartments as needed)...


            OpenWQ_couplercalls.InitialConfig(
                OpenWQ_hostModelconfig,
                OpenWQ_json,                    // create OpenWQ_json object
                OpenWQ_wqconfig,            // create OpenWQ_wqconfig object
                OpenWQ_units,                  // functions for unit conversion
                OpenWQ_readjson,            // read json files
                OpenWQ_vars,
                OpenWQ_initiate,            // initiate modules
                OpenWQ_watertransp,      // transport modules
                OpenWQ_chem,                   // biochemistry modules
                OpenWQ_sinksource,        // sink and source modules)
                OpenWQ_output);

        }

        // ################################
        // Get Variables from other modules or meteorology
        // ################################
        // SWE
        declgetvar("*", "SWE", "(mm)", &SWE);
        declputvar("*", "SWE_conc", "(mg/l)", &SWE_conc, &SWE_conc_lay);
        // runoff
        declgetvar("*", "soil_runoff", "(mm)", &soil_runoff);
        declputvar("*", "soil_runoff_cWQ", "(mm)", &soil_runoff_cWQ,&soil_runoff_cWQ_lay);
        // ssr
        declgetvar("*", "soil_ssr", "(mm)", &soil_ssr);
        declputvar("*", "soil_ssr_conc", "(mm)", &soil_ssr_conc,&soil_ssr_conc_lay);
        // Sd
        declstatvar("Sd", NHRU, "Depression storage.", "(mm)", &Sd);
        declstatvar("Sd_conc", NDEFN, "Concentration: Depression storage.", "(mg/l)", &Sd_conc, &Sd_conc_lay, Global::numsubstances);
        // soil_rechr
        declgetvar("*", "soil_rechr", "(mm)", &soil_rechr);
        declputvar("*", "conc_soil_rechr", "(mg/l)", &conc_soil_rechr, &conc_soil_rechr_lay);
        // soil moist (vol) & soil_lower (conc)
        declgetvar("*", "soil_moist", "(mm)", &soil_moist);
        declputvar("*", "conc_soil_lower", "(mg/l)", &conc_soil_lower, &conc_soil_lower_lay);
        // surficial soil (only mass)
        declputvar("*", "surfsoil_solub_mWQ", "(mg/l)", &surfsoil_solub_mWQ, &surfsoil_solub_mWQ_lay);
        // gw
        declstatvar("gw", NHRU, "ground water storage.", "(mm)", &gw);
        declstatvar("gw_conc", NDEFN, "Concentration: ground water storage.", "(mg/l)", &gw_conc, &gw_conc_lay, Global::numsubstances);

        // ################################
        // Get Parameters from other modules or meteorology
        // ################################
        // air temperature
        declgetvar("*", "hru_t", "(°C)", &hru_t);
        declparam("hru_area", NHRU, "[1]", "1e-6", "1e+09", "hru area", "(km^2)", &hru_area);
        declparam("soil_rechr_max", NHRU, "[60.0]", "0.0", "350.0",
        "Maximum value for soil recharge zone (upper portion of soil_moist where losses occur as both evaporation "//
        "and transpiration).  Must be less than or equal to soil_moist.","( )", &soil_rechr_max);


    }

    //************************************************************************************************
    // Declare and initiate variables
    // function that calls coupler functions:
    // 1) OpenWQ_couplercalls.RunTimeLoopStart
    // 2) OpenWQ_couplercalls.RunSpaceStep
    // 3) OpenWQ_couplercalls.RunTimeLoopEnd
    //************************************************************************************************
    void CLASSWQ_openwq::run(
        OpenWQ_couplercalls& OpenWQ_couplercalls,
        OpenWQ_hostModelconfig& OpenWQ_hostModelconfig,
        OpenWQ_json& OpenWQ_json,                    // create OpenWQ_json object
        OpenWQ_wqconfig& OpenWQ_wqconfig,            // create OpenWQ_wqconfig object
        OpenWQ_units& OpenWQ_units,                  // functions for unit conversion
        OpenWQ_readjson& OpenWQ_readjson,            // read json files
        OpenWQ_vars& OpenWQ_vars,
        OpenWQ_initiate& OpenWQ_initiate,            // initiate modules
        OpenWQ_watertransp& OpenWQ_watertransp,      // transport modules
        OpenWQ_chem& OpenWQ_chem,                    // biochemistry modules
        OpenWQ_sinksource& OpenWQ_sinksource,        // sink and source modules)
        OpenWQ_solver& OpenWQ_solver,                // solver module
        OpenWQ_output& OpenWQ_output)                // output modules
        {

        // Local Variables
        unsigned int Sub_mob;   // interactive species index (for mobile species)

        // Retrieve simulation timestamp
        // convert to OpenWQ time convention: seconds since 00:00 hours, Jan 1, 1970 UTC
        // this allows the use of a number of funtions of C++

        time_t simtime = (Global::DTnow - 70 * 365 - 21) * 24 * 3600;

        // Update Dependencies to kinetic formulas (needs loop to get hydro model variables
        // that are dependencies to OpenWQ)
        #pragma omp parallel for num_threads(OpenWQ_wqconfig.num_threads_requested)
        for (unsigned int hru=0;hru<nhru;hru++){
            (*OpenWQ_hostModelconfig.SM)(hru,0,0) = soil_rechr[hru]/soil_rechr_max[hru];  // loop needed - Save all SM data from hostmodel at time t
            (*OpenWQ_hostModelconfig.Tair)(hru,0,0) = hru_t[hru];  // loop needed - Save all Tair data from hostmodel at time t
            (*OpenWQ_hostModelconfig.Tsoil)(hru,0,0) = hru_t[hru];   // keeping the same as Tair for now
        }


        // Get Fluxes from Hydrological model
        // Save them to (*OpenWQ_hostModelconfig.waterVol_hydromodel)
        // Convert units to m3

        // CRHM water units are in mm
        // CRHM area units are in km2
        // So conversion of mm (water) to m3 is calculated as follows:
        // water_m3 = (water_mm / 1000) * (area_km2 * 1000000) <=>
        // water_m3 = water_mm * area_km2 * 1000
        #pragma omp parallel for num_threads(OpenWQ_wqconfig.num_threads_requested)
        for (unsigned int hru=0;hru<nhru;hru++){

            // SWE
            (*OpenWQ_hostModelconfig.waterVol_hydromodel)[0](hru,0,0) = std::fmax(SWE[hru] * 1000 * hru_area[hru], 0.0f);
            // soil_runoff
            (*OpenWQ_hostModelconfig.waterVol_hydromodel)[1](hru,0,0) = std::fmax(soil_runoff[hru] * 1000 * hru_area[hru], 0.0f);
            // soil ssr
            (*OpenWQ_hostModelconfig.waterVol_hydromodel)[2](hru,0,0) = std::fmax(soil_ssr[hru] * 1000 * hru_area[hru], 0.0f);
            // Sd
            (*OpenWQ_hostModelconfig.waterVol_hydromodel)[3](hru,0,0) = std::fmax(Sd[hru] * 1000 * hru_area[hru], 0.0f);
            // Soi_rechr
            (*OpenWQ_hostModelconfig.waterVol_hydromodel)[4](hru,0,0) = std::fmax(soil_rechr[hru] * 1000 * hru_area[hru], 0.0f);
            // Soil_moist
            (*OpenWQ_hostModelconfig.waterVol_hydromodel)[5](hru,0,0) = std::fmax((soil_moist[hh] - soil_rechr[hh]) * 1000 * hru_area[hru] , 0.0f);
            // Surfsoil (not water in it)
            (*OpenWQ_hostModelconfig.waterVol_hydromodel)[6](hru,0,0) = 1;
            // gw
            (*OpenWQ_hostModelconfig.waterVol_hydromodel)[7](hru,0,0) = std::fmax(gw[hru] * 1000 * hru_area[hru], 0.0f);

        }

        // ##############################################
        // Calls all functions required
        // INSIDE the TIME loop
        // But BEFORE the SPACE loop is initiated
        // ##############################################
        OpenWQ_couplercalls.RunTimeLoopStart(
            OpenWQ_hostModelconfig,
            OpenWQ_json,                    // create OpenWQ_json object
            OpenWQ_wqconfig,            // create OpenWQ_wqconfig object
            OpenWQ_units,                  // functions for unit conversion
            OpenWQ_readjson,            // read json files
            OpenWQ_vars,
            OpenWQ_initiate,            // initiate modules
            OpenWQ_watertransp,      // transport modules
            OpenWQ_chem,                   // biochemistry modules
            OpenWQ_sinksource,        // sink and source modules)
            OpenWQ_solver,
            OpenWQ_output,
            simtime);


        // ##############################################
        // Calls all functions required
        // INSIDE the SPACE loop
        // TRANSPORT routines
        // ##############################################

        // Initiation steps that only need to be done once (at the start of the simulation)
        if (OpenWQ_hostModelconfig.interaction_step > 1){

            // ################################
            // Retried current state of state variables form CRHM
            // Convert from concentration (default unit in CRHM) to mass (default unit of OpenWQ)
            // ################################

            // iteractive compartment index
            unsigned int icmp;

            #pragma omp parallel for private(Sub_mob) num_threads(OpenWQ_wqconfig.num_threads_requested)
            for(unsigned int hru = 0; hru < nhru; ++hru) {

                for(unsigned int Sub_i = 0; Sub_i < OpenWQ_wqconfig.BGC_general_mobile_species.size(); ++Sub_i) {

                    // ################################
                    // Adv-Disp directly from CRHM's modules

                    // Get mobile species indexes
                    // Only update mass/concentrations using CRHM's water for mobile species
                    Sub_mob = OpenWQ_wqconfig.BGC_general_mobile_species[Sub_i];

                    // SWE
                    (*OpenWQ_vars.d_chemass_dt_transp)(0)(Sub_mob)(hru,0,0) =
                        (std::fmax(SWE_conc_lay[Sub_mob][hru],0.0f)
                            * (*OpenWQ_hostModelconfig.waterVol_hydromodel)[0](hru,0,0)
                        ) - (*OpenWQ_vars.chemass)(0)(Sub_mob)(hru,0,0);

                    // RUNOFF
                    (*OpenWQ_vars.d_chemass_dt_transp)(1)(Sub_mob)(hru,0,0) =
                        (std::fmax(soil_runoff_cWQ_lay[Sub_mob][hru],0.0f)
                        * (*OpenWQ_hostModelconfig.waterVol_hydromodel)[1](hru,0,0)
                        ) - (*OpenWQ_vars.chemass)(1)(Sub_mob)(hru,0,0);

                    // SSR
                    (*OpenWQ_vars.d_chemass_dt_transp)(2)(Sub_mob)(hru,0,0) =
                        (std::fmax(soil_ssr_conc_lay[Sub_mob][hru],0.0f)
                        * (*OpenWQ_hostModelconfig.waterVol_hydromodel)[2](hru,0,0)
                        ) - (*OpenWQ_vars.chemass)(2)(Sub_mob)(hru,0,0);

                    // SD
                    (*OpenWQ_vars.d_chemass_dt_transp)(3)(Sub_mob)(hru,0,0) =
                        (std::fmax(Sd_conc_lay[Sub_mob][hru],0.0f)
                        * (*OpenWQ_hostModelconfig.waterVol_hydromodel)[3](hru,0,0)
                        ) - (*OpenWQ_vars.chemass)(3)(Sub_mob)(hru,0,0);


                    // SOIL_RECHR
                    (*OpenWQ_vars.d_chemass_dt_transp)(4)(Sub_mob)(hru,0,0) =
                        (std::fmax(conc_soil_rechr_lay[Sub_mob][hru],0.0f)
                        * (*OpenWQ_hostModelconfig.waterVol_hydromodel)[4](hru,0,0)
                        ) - (*OpenWQ_vars.chemass)(4)(Sub_mob)(hru,0,0);

                    // SOIL LOWER
                    (*OpenWQ_vars.d_chemass_dt_transp)(5)(Sub_mob)(hru,0,0) =
                        (std::fmax(conc_soil_lower_lay[Sub_mob][hru],0.0f)
                        * (*OpenWQ_hostModelconfig.waterVol_hydromodel)[5](hru,0,0)
                        ) - (*OpenWQ_vars.chemass)(5)(Sub_mob)(hru,0,0);

                    // SURFSOIL
                    // (does not multiply by volume of water because CRHM uses mass
                    //for this specific compartment)
                    (*OpenWQ_vars.d_chemass_dt_transp)(6)(Sub_mob)(hru,0,0) =
                        (std::fmax(surfsoil_solub_mWQ_lay[Sub_mob][hru],0.0f)
                        ) - (*OpenWQ_vars.chemass)(6)(Sub_mob)(hru,0,0);


                    // GW
                    (*OpenWQ_vars.d_chemass_dt_transp)(7)(Sub_mob)(hru,0,0) =
                        (std::fmax(gw_conc_lay[Sub_mob][hru],0.0f)
                        * (*OpenWQ_hostModelconfig.waterVol_hydromodel)[7](hru,0,0)
                        ) - (*OpenWQ_vars.chemass)(7)(Sub_mob)(hru,0,0);

                }

                // ################################
                // EROSION: Called to calculation erosion
                // Transport is calculated by native CRHM routines
                // only need to consider

                // Runoff
                icmp = 1;
                OpenWQ_couplercalls.RunSpaceStep(
                    OpenWQ_hostModelconfig,
                    OpenWQ_json,                    // create OpenWQ_json object
                    OpenWQ_wqconfig,            // create OpenWQ_wqconfig object
                    OpenWQ_units,                  // functions for unit conversion
                    OpenWQ_readjson,            // read json files
                    OpenWQ_vars,
                    OpenWQ_initiate,            // initiate modules
                    OpenWQ_watertransp,      // transport modules
                    OpenWQ_chem,                   // biochemistry modules
                    OpenWQ_sinksource,        // sink and source modules)
                    OpenWQ_solver,
                    OpenWQ_output,
                    simtime,                            // simulation time in seconds since seconds since 00:00 hours, Jan 1, 1970 UTC
                    icmp,      // source: runoff
                    hru,    // x
                    0,      // y
                    0,      // z
                    icmp,      // receipient: runoff (the same as source because for CRHM we'll only deal with erosion, and for simplicity we'll just put it in the same compartment)
                    hru,    // x
                    0,      // y
                    0,      // z
                    soil_runoff[hru] * 1000 * hru_area[hru],          // water flux from source compartment to receipient compartment
                    (*OpenWQ_hostModelconfig.waterVol_hydromodel)[icmp](hru,0,0));   // water mass of the receipient compartment

                // SSR
                icmp = 2;
                OpenWQ_couplercalls.RunSpaceStep(
                    OpenWQ_hostModelconfig,
                    OpenWQ_json,                    // create OpenWQ_json object
                    OpenWQ_wqconfig,            // create OpenWQ_wqconfig object
                    OpenWQ_units,                  // functions for unit conversion
                    OpenWQ_readjson,            // read json files
                    OpenWQ_vars,
                    OpenWQ_initiate,            // initiate modules
                    OpenWQ_watertransp,      // transport modules
                    OpenWQ_chem,                   // biochemistry modules
                    OpenWQ_sinksource,        // sink and source modules)
                    OpenWQ_solver,
                    OpenWQ_output,
                    simtime,                            // simulation time in seconds since seconds since 00:00 hours, Jan 1, 1970 UTC
                    icmp,      // source: runoff
                    hru,    // x
                    0,      // y
                    0,      // z
                    icmp,      // receipient: runoff (the same as source because for CRHM we'll only deal with erosion, and for simplicity we'll just put it in the same compartment)
                    hru,    // x
                    0,      // y
                    0,      // z
                    soil_ssr[hru] * 1000 * hru_area[hru],          // water flux from source compartment to receipient compartment
                    (*OpenWQ_hostModelconfig.waterVol_hydromodel)[icmp](hru,0,0));   // water mass of the receipient compartment


            }

        }

       // ##############################################
        // Calls all functions required
        // INSIDE the TIME loop
        // But AFTER the SPACE loop is initiated
        // ##############################################
       OpenWQ_couplercalls.RunTimeLoopEnd(
           OpenWQ_hostModelconfig,
                OpenWQ_json,                    // create OpenWQ_json object
                OpenWQ_wqconfig,            // create OpenWQ_wqconfig object
                OpenWQ_units,                  // functions for unit conversion
                OpenWQ_readjson,            // read json files
                OpenWQ_vars,
                OpenWQ_initiate,            // initiate modules
                OpenWQ_watertransp,      // transport modules
                OpenWQ_chem,                   // biochemistry modules
                OpenWQ_sinksource,        // sink and source modules)
                OpenWQ_solver,
                OpenWQ_output,
                simtime);

        //}

        // ################################
        // Save the arma::variable results back to CRHM's variables (for each compartment)
        // Convert from mass (default unit in OpenWQ) to concentrations (default unit of CRHM-WQ)
        // ################################

        // Water volume conversions
            // CRHM water units are in mm
            // CRHM area units are in km2
            // So conversion of mm (water) to m3 is calculated as follows:
            // water_m3 = (water_mm / 1000) * (area_km2 * 1000000) <=>
            // water_m3 = water_mm * area_km2 * 1000


        #pragma omp parallel for private(Sub_mob) collapse(2) num_threads(OpenWQ_wqconfig.num_threads_requested)
        for(unsigned int hru = 0; hru < nhru; ++hru) {

            for(unsigned int Sub_i = 0; Sub_i < OpenWQ_wqconfig.BGC_general_mobile_species.size(); ++Sub_i) {

                    // Get mobile species indexes
                    // Only update mass/concentrations using CRHM's water for mobile species
                    Sub_mob = OpenWQ_wqconfig.BGC_general_mobile_species[Sub_i];

                // SWE
                if ((*OpenWQ_hostModelconfig.waterVol_hydromodel)[0](hru,0,0)
                        > OpenWQ_hostModelconfig.watervol_minlim){

                    SWE_conc_lay[Sub_mob][hru] =
                        (*OpenWQ_vars.chemass)(0)(Sub_mob)(hru,0,0)
                        / (*OpenWQ_hostModelconfig.waterVol_hydromodel)[0](hru,0,0);

                }else{
                    SWE_conc_lay[Sub_mob][hru] = 0.0f;
                }

                // RUNOFF
                if ((*OpenWQ_hostModelconfig.waterVol_hydromodel)[1](hru,0,0)
                        > OpenWQ_hostModelconfig.watervol_minlim){

                    soil_runoff_cWQ_lay[Sub_mob][hru] =
                        (*OpenWQ_vars.chemass)(1)(Sub_mob)(hru,0,0)
                        / (*OpenWQ_hostModelconfig.waterVol_hydromodel)[1](hru,0,0);

                }else{
                    soil_runoff_cWQ_lay[Sub_mob][hru] = 0.0f;
                }

                // SSR
                if ((*OpenWQ_hostModelconfig.waterVol_hydromodel)[2](hru,0,0)
                        > OpenWQ_hostModelconfig.watervol_minlim){

                    soil_ssr_conc_lay[Sub_mob][hru]=
                        (*OpenWQ_vars.chemass)(2)(Sub_mob)(hru,0,0)
                        / (*OpenWQ_hostModelconfig.waterVol_hydromodel)[2](hru,0,0);

                }else{
                    soil_ssr_conc_lay[Sub_mob][hru] = 0.0f;
                }

                // SD
                if ((*OpenWQ_hostModelconfig.waterVol_hydromodel)[3](hru,0,0)
                        > OpenWQ_hostModelconfig.watervol_minlim){

                    Sd_conc_lay[Sub_mob][hru] =
                        (*OpenWQ_vars.chemass)(3)(Sub_mob)(hru,0,0)
                        / (*OpenWQ_hostModelconfig.waterVol_hydromodel)[3](hru,0,0);

                }else{
                    Sd_conc_lay[Sub_mob][hru] = 0.0f;
                }

                // SOIL_RECHR
                if ((*OpenWQ_hostModelconfig.waterVol_hydromodel)[4](hru,0,0)
                        > OpenWQ_hostModelconfig.watervol_minlim){

                    conc_soil_rechr_lay[Sub_mob][hru] =
                        (*OpenWQ_vars.chemass)(4)(Sub_mob)(hru,0,0)
                        / (*OpenWQ_hostModelconfig.waterVol_hydromodel)[4](hru,0,0);

                }else{
                    conc_soil_rechr_lay[Sub_mob][hru] = 0.0f;
                }

                // SOIL LOWER
                if ((*OpenWQ_hostModelconfig.waterVol_hydromodel)[5](hru,0,0)
                        > OpenWQ_hostModelconfig.watervol_minlim){

                    conc_soil_lower_lay[Sub_mob][hru] =
                        (*OpenWQ_vars.chemass)(5)(Sub_mob)(hru,0,0)
                        / (*OpenWQ_hostModelconfig.waterVol_hydromodel)[5](hru,0,0);;

                }else{
                    conc_soil_lower_lay[Sub_mob][hru] = 0.0f;
                }

                // SURFSOIL
                // (does not divide by volume of water because CRHM uses mass
                //for this specific compartment)
                surfsoil_solub_mWQ_lay[Sub_mob][hru] = (*OpenWQ_vars.chemass)(6)(Sub_mob)(hru,0,0);

                // GW
                if ((*OpenWQ_hostModelconfig.waterVol_hydromodel)[7](hru,0,0)
                        > OpenWQ_hostModelconfig.watervol_minlim){

                    gw_conc_lay[Sub_mob][hru] =
                        (*OpenWQ_vars.chemass)(7)(Sub_mob)(hru,0,0)
                        / (*OpenWQ_hostModelconfig.waterVol_hydromodel)[7](hru,0,0);

                }else{
                    gw_conc_lay[Sub_mob][hru] = 0.0f;
                }

            }
        }

    }

